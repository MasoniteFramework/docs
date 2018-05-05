# Creating a Mail Driver

## Introduction

Because of Masonite's Service Container, It is extremely easy to make drivers that can be used by simply adding your service provider.

## Getting Started

Masonite comes shipped with a Service Provider called `MailProvider` which loads a few classes into the container as well as boots the default mail driver using the `MailManager`. This manager class will fetch drivers from the container and instantiate them. We can look at the `MailProvider` class which will gives us a better explanation as to what's going on:

```python
class MailProvider(ServiceProvider):

    wsgi = False

    def register(self):
        self.app.bind('MailConfig', mail)
        self.app.bind('MailSmtpDriver', MailSmtpDriver)
        self.app.bind('MailMailgunDriver', MailMailgunDriver)

    def boot(self):
        self.app.bind('Mail', MailManager(self.app))
```

We can see here that because we are only binding things into the container and we don't need the WSGI server to be running, we set `wsgi = False`. Service Providers that set `wsgi` to `False` will only run when the server starts and not on every request.

We can see here that we are binding a few drivers into the container and then binding the `MailManager` on boot. Remember that our boot method has access to everything that has been registered into the container. The register methods are executed on all providers before the boot methods are executed.

### Mail Manager

The `MailManager` here is important to understand. When the `MailManager` is instantiated, it accepts the container as a parameter. When the `MailManager` is instantiated, it fires a `create_driver` method which will grab the driver from the configuration file and retrieve a `MailXDriver` from the container. The `create_driver` method is a very simple method:

```python
def create_driver(self, driver=None):
    if not driver:
        driver = self.container.make('MailConfig').DRIVER.capitalize()
    else:
        driver = driver.capitalize()

    try:
        self.manage_driver = self.container.make('Mail{0}Driver'.format(driver))
    except KeyError:
        raise DriverNotFound('Could not find the Mail{0}Driver from the service container. Are you missing a service provider?'.format(driver))
```

Notice that when the driver is created, it tries to get a `Mail{0}Driver` from the container. Therefore, all we need to do is register a `MailXDriver` into the container \('X' being the name of the driver\) and Masonite will know to grab that driver.

### Creating a Driver

So now we know that we need a `MailXDriver` so let's walk through how we could create a `maildrill` email driver.

We can simply create a class which can become our driver. We do not need to inherit anything, although Masonite comes with a `BaseMailDriver` to get you started faster and all drivers should inherit from it for consistency reasons. You can make your driver from a normal class object but it will be harder and won't be considered in Pull Requests.

Let's create a class anywhere we like and inherit from `BaseMailDriver`:

```python
from masonite.driver.BaseMailDriver import BaseMailDriver

class MailMaildrillDriver(BaseMailDriver):
    pass
```

Great! We are almost done. We just have to implement one method on this class and that's the `send` method. All other methods like `to` and `template` are inherited from the `BaseMailDriver` class. You can find out how to send an email using Maildrill and implement it in this `send` method.

We can look at other drivers for inspiration but let's look at the `MailMailgunDriver` class now:

```python
import requests
from masonite.drivers.BaseMailDriver import BaseMailDriver

class MailMailgunDriver(BaseMailDriver):

    def send(self, message=None):
        if not message:
            message = self.message_body

        domain = self.config.DRIVERS['mailgun']['domain']
        secret = self.config.DRIVERS['mailgun']['secret']
        return requests.post(
            "https://api.mailgun.net/v3/{0}/messages".format(domain),
            auth=("api", secret),
            data={"from": "{0} <mailgun@{1}>".format(self.config.FROM['name'], domain),
                  "to": [self.to_address, "{0}".format(self.config.FROM['address'])],
                "subject": self.message_subject,
                "html": message})
```

If you are wondering where the `self.message_body` and `self.config` are coming from, check the `BaseMailDriver`. All driver constructors are resolved by the service container so you can grab anything you need from the container to make your driver work. Notice here that we don't need a constructor because we inherited it from the `BaseMailDriver`

### Registering Your Mail Driver

Since the `MailManager` class creates the driver on boot, we can simply register the driver into the container via any service providers register method. We could create a new Service Provider and register it there. You can read more about created Service Providers under the [Service Providers](../architectural-concepts/service-providers.md) documentation. For now, we will just register it from within our `AppProvider`.

Our `AppProvider` class might look something like this:

```python
from your.driver.module import MailMandrillDriver
class AppProvider(ServiceProvider):

    wsgi = True

    def register(self):
        self.app.bind('WebRoutes', web.ROUTES)
        self.app.bind('ApiRoutes', api.ROUTES)
        self.app.bind('Response', None)
        self.app.bind('Storage', storage)

        # Register new mail driver
        self.app.bind('MailMandrillDriver', MailMandrillDriver)

    def boot(self, Environ):
        self.app.bind('Request', Request(Environ))
        self.app.bind('Route', Route(Environ))
```

Great! Our new driver is registered into the container. It is now able to be created with Masonite's `MailManager` class. We can retrieve your new driver by doing:

```python
def show(self, Mail)
    Mail.driver('mandrill') # fetches MailMandrillDriver from container
```

### Configuration

If we want the `MailManager` to use our new driver by default, change the `DRIVER` in our `config/mail.py` file. In addition, you may have the users of your driver require a special dictionary entry to the `DRIVERS` dictionary:

```python
DRIVERS = {
    'smtp': {
        'host': os.getenv('MAIL_HOST', 'smtp.mailtrap.io'),
        'port': os.getenv('MAIL_PORT', '465'),
        'username': os.getenv('MAIL_USERNAME', 'username'),
        'password': os.getenv('MAIL_PASSWORD', 'password'),
    },
    'mailgun': {
        'secret': os.getenv('MAILGUN_SECRET', 'key-XX'),
        'domain': os.getenv('MAILGUN_DOMAIN', 'sandboxXX.mailgun.org')
    },
    'maildrill': {
        'secret': 'xx'
        'other_key': 'xx'
    }
}
```

This way, users can easily swap drivers by simply changing the driver in the config file.

That's it! We just extended our Masonite project and created a new driver. Consider making it available on PyPi so others can install it!

