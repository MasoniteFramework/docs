Service Providers are a critical architecture concept in Masonite. Masonite is built around an IOC container and service providers allow you to interact with this container to add functionality to your application. Service providers run in order inside your `PROVIDERS` list.

A service provider is not limited on the features it can add your application. In fact, every aspect of Masonite is done through service providers. Everything from mail, queue and broadcasting features all the way to routing and view rendering.

In most cases you don't have to create your own service providers but if you want to extend the framework, extend a feature or create your own features, then you should 

# Getting Started

In a default project, you can find your providers in the `config/providers.py` file. This will be a collection of all providers in your application from framework providers to third party providers to application specific providers you build yourself. Anytime you add a provider you will have to register your provider in your `PROVIDERS` list.

# Creating Providers

You can create a provider using a command:

```
python craft provider YourProvider
```

This will create a provider that looks like this:

```python
from masonite.providers import Provider

class YourProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        pass

    def boot(self):
        pass
```

Providers have 2 methods: `register` and `boot`.

The register method is called when the provider is first loaded into the container. Use this to register new classes and features to your application.

The boot method is called on every request. Use this to run logic needed during a request.

# Registering Providers

You can easily add your provider to the `PROVIDERS` list in your `config/providers.py` file:

```python
from app.providers.YourProvider import YourProvider

PROVIDERS = [
  #..
  YourProvider,
  #..
]
```

