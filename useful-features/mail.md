# Mail

## Introduction

Masonite comes with email support out of the box. Most projects you make will need to send emails upon actions like account creation or notifications. Because email is used so often with software applications, masonite provides mail support with several drivers.

## Getting Started

All mail configuration is inside `config/mail.py` and contains several well documented options. There are several built in drivers you can use but you can make your own if you'd like.

{% hint style="success" %}
You can follow the documentation here at [Creating a Mail Driver](../advanced/creating-a-mail-driver.md). If you do make your own, consider making it available on PyPi so others can install it. We may even put it in Masonite by default.
{% endhint %}

By default, Masonite uses the `smtp` driver. Inside your `.env` file, just put your smtp credentials. If you are using Mailgun then switch your driver to `mailgun` and put your Mailgun credentials in your `.env` file.

## Configuring Drivers

There are two drivers out of the box that masonite uses and there is a tiny bit of configuration for both.

### SMTP Driver

The SMTP driver takes several configuration files we can all put in our `.env` file.

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
MAIL_DRIVER=smtp
MAIL_FROM_ADDRESS=admin@email.com
MAIL_FROM_NAME=Masonite
MAIL_HOST=smtp.gmail.com
MAIL_PORT=465
MAIL_USERNAME=admin@email.com
MAIL_PASSWORD=password
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Because this is SMTP, we can utilize all SMTP services such as mailtrap and gmail.

#### SSL \(optional\)

You may need to use an ssl version of SMTP depending on the service you are using. You can specify to use SSL by setting that option in your smtp driver configuration in `config/mail.py`:

```python
DRIVERS = {
    'smtp': {
        'host': os.getenv('MAIL_HOST', 'smtp.mailtrap.io'),
        'port': os.getenv('MAIL_PORT', '465'),
        'username': os.getenv('MAIL_USERNAME', 'username'),
        'password': os.getenv('MAIL_PASSWORD', 'password'),
        'ssl': True
    },
```

Thats it! As long as the authentication works, we can send emails.

{% hint style="danger" %}
Remember that it is save to put sensitive data in your `.env` file because it is not committed to source control and it is inside the `.gitignore` file by default.
{% endhint %}

### Mailgun Driver

Mailgun does not use SMTP and instead uses API calls to their service to send emails. Mailgun only requires 2 configuration settings:

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
MAILGUN_SECRET=key-xx
MAILGUN_DOMAIN=sandboxXX.mailgun.org
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If you change to using Mailgun then you will need to change the driver. By default the driver looks like:

{% code-tabs %}
{% code-tabs-item title="config/mail.py" %}
```python
DRIVER = os.getenv('MAIL_DRIVER', 'smtp')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This means you can specify the mail driver in the .env file:

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
MAIL_DRIVER=mailgun
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or we can specify the driver directly inside `config/mail.py`

{% code-tabs %}
{% code-tabs-item title="config/mail.py" %}
```python
DRIVER = 'mailgun'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Masonite will retrieve the configuration settings for the mailgun driver from the `DRIVERS` configuration setting which Masonite has by default, you do not have to change this.

{% code-tabs %}
{% code-tabs-item title="config/mail.py" %}
```python
DRIVERS = {
    ...
    'mailgun': {
        'secret': os.getenv('MAILGUN_SECRET', 'key-XX'),
        'domain': os.getenv('MAILGUN_DOMAIN', 'sandboxXX.mailgun.org')
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Terminal Driver

The Terminal driver simply prints out your email message in the terminal. Makes testing and development super easy. To use the terminal driver you'll need to enter a few configuration settings.

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
MAIL_DRIVER=terminal
MAIL_FROM_ADDRESS=admin@email.com
MAIL_FROM_NAME=Masonite
MAIL_HOST=
MAIL_PORT=
MAIL_USERNAME=
MAIL_PASSWORD=
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Log Driver

The Log driver simply prints out your email message into a log file. To use the log driver you'll need to enter a few configuration settings.

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
MAIL_DRIVER=log
MAIL_FROM_ADDRESS=admin@email.com
MAIL_FROM_NAME=Masonite
MAIL_HOST=
MAIL_PORT=
MAIL_USERNAME=
MAIL_PASSWORD=
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Masonite will retrieve the configuration settings for the log driver from the `DRIVERS` configuration setting which Masonite has by default, you do not have to change this.

{% code-tabs %}
{% code-tabs-item title="config/mail.py" %}
```python
DRIVERS = {
    ...
    'log': {
        'file': os.getenv('LOG_FILE', 'mail.log'),
        'location': 'bootstrap/logs'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Sending an Email

The `Mail` class is loaded into the container via the the `MailProvider` Service Provider. We can fetch this `Mail` class via our controller methods:

```python
from masonite import Mail

def show(self, mail: Mail):
    print(mail) # returns the default mail driver
```

We can send an email like so:

```python
from masonite import Mail

def show(self, mail: Mail):
    mail.to('hello@email.com').send('Welcome!')
```

You can also obviously specify a specific user:

```python
from app.User import User
from masonite import Mail
...
def show(self, mail: Mail):
    mail.to(User.find(1).email).send('Welcome!')
```

## Switching Drivers

All mail drivers are managed by the `MailManager` class and bootstrapped with the `MailProvider` Service Provider.

We can specify which driver we want to use. Although Masonite will use the `DRIVER` variable in our `mail` config file by default, we can change the driver on the fly.

You can see in our `MailProvider` Service Provider that we can use the `MailManager` class to set the driver. We can use this same class to change the driver:

```python
from masonite.manager import MailManager

def show(self, manager: MailManager):
    manager.driver('mailgun') # now uses the Mailgun driver
```

## Queues

Sending an email may take several seconds so it might be a good idea to create a Job. Jobs are simply Python classes that inherit from the `Queueable` class and can be pushed to queues or ran asynchronously. This will look something like:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail
from masonite import Queue

def show(self, queue: Queue):
    queue.push(SendWelcomeEmail)
```

Instead of taking seconds to send an email, this will seem immediate and be sent using whatever queue driver is set. The `async` driver is set by default which requires no additional configuration and simply sends jobs into a new thread to be ran in the background.

{% hint style="success" %}
Read more about creating Jobs and sending emails asynchronously in the [Queues and Jobs](queues-and-jobs.md) documentation.
{% endhint %}

## Methods

We can also specify the subject:

```python
mail.subject('Welcome!').to('hello@email.com').send('Welcome!')
```

You can specify which address you want the email to appear from:

```python
mail.send_from('Admin@email.com').to('hello@email.com').send('Welcome!')
```

## Templates

If you don't want to pass a string as the message, you can pass a view template.

```python
mail.to('idmann509@gmail.com').template('mail/welcome').send()
```

This will render the view into a message body and send the email as html. Notice that we didn't pass anything into the `send` message

### Passing Data to Templates

You are also able to pass data into our mail templates. This data is passed in as a dictionary that contains a key which is the variable with the corresponding value. We can pass data to the function like so:

```python
mail.to('idmann509@gmail.com').template('mail/welcome', {'name': 'Masonite User'}).send()
```

