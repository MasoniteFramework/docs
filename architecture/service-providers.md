Service Providers are the key building blocks to Masonite. The only thing they do is register things into the [Service Container](service-container.md), or run logic on requests. If you look inside the `config/providers.py` file, you will find a `PROVIDERS` list which contains all the Service Providers involved in building the framework.

Although uncommon, You may create your own service provider and add it to your providers list to extend Masonite, or even remove some providers if you don't need their functionality. If you do create your own Service Provider, consider making it available on PyPi so others can install it into their framework.

# Creating a Provider

We can create a Service Provider by simply using a craft command:

```text
$ python craft provider DashboardProvider
```

This will create a new Service Provider under our `app/providers/DashboardProvider.py`. This new Service Provider will have two simple methods, a `register` method and a `boot` method. We'll explain both in detail.

# Service Provider Execution

There are a few architectural examples we will walk through to get you familiar with how Service Providers work under the hood. Let's look at a simple provider and walk through it.

```python
from masonite.providers import Provider


class YourProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.bind('User', User)

    def boot(self):
        print(self.application.make('User'))
```

We can see that we have a simple provider that registers the `User` model into the container. There are three key features we have to go into detail here.


## Register

In our `register` method, it's important that we only bind things into the container. When the provider is first registered to the container, the register method is ran and your classes will be registered to the container.

## Boot

The boot method will have access to everything that is registered in the container. The boot method is ran during requests and is actually resolved by the container. Because of this, we can actually rewrite our provider above as this:

```python
from masonite.provider import ServiceProvider
from app.User import User

class UserModelProvider(ServiceProvider):
    ''' Binds the User model into the Service Container '''

    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.bind('User', User)

    def boot(self, user: User):
        print(user)
```

> This will be exactly the same as above. Notice that the `boot` method is resolved by the container.

