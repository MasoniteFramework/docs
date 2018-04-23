# Service Providers

## Introduction

Service Providers are the key building blocks to Masonite. The only thing they do is register things into the Service Container, or retrieve things from the Service Container. You can read more about the Service Container in the [Service Container](service-container.md) documentation. If you look inside the `config/application.py` file, you will find a `PROVIDERS` list which contains all the Service Providers involved in building the framework.

You may create your own service provider and add it to your providers list to extend Masonite, or even remove some providers if you don't need their functionality. If you do create your own Service Provider, consider making it available on PyPi so others can install it into their framework.

## Creating a Provider

We can create a Service Provider but simply using a craft command:

```text
$ craft provider DashboardProvider
```

This will create a new Service Provider under our `app/providers/DashboardProvider.py`. This new Service Provider will have two simple methods, a `register` method and a `boot` method. We'll explain both in detail.

## Service Provider Execution

There are a few architectural examples we will walk through to get you familiar with how Service Providers work under the hood. Let's look at a simple provider and walk through it.

```python
from masonite.provider import ServiceProvider
from app.User import User

class UserModelProvider(ServiceProvider):
    ''' Binds the User model into the Service Container '''

    wsgi = False 

    def register(self):
        self.app.bind('User', User)

    def boot(self):
        print(self.app.make('User'))
```

We can see that we have a simple provider that registers the `User` model into the container. There are three key features we have to go into detail here.

## WSGI

First, the `wsgi = False` just tells Masonite that this specific provider does not need the WSGI server to be running. When the WSGI server first starts, it will execute all service providers that have `wsgi` set to `False`. Whenever a provider only binds things into the container and we don't need things like requests or routes, then consider setting `wsgi` to `False`. the `ServiceProvider` class we inherited from sets `wsgi` to `True` by default. Whenever `wsgi` is `True` then the service provider will fire on every request.

## Register

In our `register` method, it's important that we only bind things into the container. When the server is booted, Masonite will execute all register methods on all service providers. This is so the `boot` method will have access to the entire container.

## Boot

The boot method will have access to everything that is registered in the container and is actually resolved by the container. Because of this, we can actually rewrite our provider above as this:

```python
from masonite.provider import ServiceProvider
from app.User import User

class UserModelProvider(ServiceProvider):
    ''' Binds the User model into the Service Container '''

    wsgi = False 

    def register(self):
        self.app.bind('User', User)

    def boot(self, User):
        print(User)
```

This will be exactly the same as above. Notice that the `boot` method is resolved by the container.

Great! It's really that simple. Just this knowledge will take you a long way. Take a look at the other service providers to get some inspiration on how you should create yours. Again, if you do create a Service Provider, consider making it available on PyPi so others can install it into their framework.

