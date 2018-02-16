# Service Container

# Introduction

The Service Container is an extremely powerful feature of Masonite and should be used to the fullest extent possible. It's important to understand the concepts of the Service Container. It's a simple concept but is a bit magical if you don't understand what's going on under the hood.

## Getting Started

The Service Container is just a container where classes are loaded into it by key-value pairs, and then can be retrieved by either the key or value. That's it.

The container is contained inside the `App` class which is instantiated throughout in the beginning of the framework and passed through various parts of the project such as controllers, middleware and drivers.

We can create easily create service providers using a craft command:


$ craft provider DashboardProvider
This will create a new service provider in `app/providers/DashboardServiceProvider.py`. Inside our new service provider we will have a `register` method and a `boot` method. It's important to know what each one does. We'll discuss both in a little bit.

### Configuration

Once you create your service provider, you'll have to add it to your `PROVIDERS` list inside your `config/application.py` file. This should be a string to the location of the class:

```python
PROVIDERS = [
    'app.providers.UserModelProvider.UserModelProvider'
    'app.providers.DashboardProvider.DashboardProvider'
]
```

### Register

The `register` methods on all service providers are executed first, then all the `boot` methods are executed after. Because of this, the `register` method should only be used to **load things into the container**.

### Boot

The boot method is executed on all service providers after the `register` methods on all service providers have been called. Because of this, the boot method will have access to everything inside the container and is resolved by Masonite's container.

### Binding

In order to bind classes into the container, we will just need to use a simple `bind` method on our `app` container. In a service provider, that will look like:

```python
from masonite.provider import ServiceProvider
from app.User import User
from masonite.request import Request

class UserModelProvider(ServiceProvider):

    def register(self):
        self.app.bind('User', User)

    def boot(self):
        pass

```

This will load the key value pair in the `providers` dictionary in the container. The dictionary after this call will look like:

```
>>> app.providers
{'User': User}
```

The service container is injected into the `Request` object and can be retrieved by:

```python
def show(self, Request):
    Request.app() # will return the service container
```

### Using the container

The container can be used in two ways: **making** and **resolving**.

#### Making

In order to retrieve a class from the service container, we can simply use the `make` method.

```
>>> from app.User import User
>>> app.bind('User', User)
>>> app.make('User')
<class app.User.User>
```

That's it! This is useful as an IOC container which you can load a single class into the container and use that class everywhere throughout your project.

#### Resolving

This is the most useful part of the container. It is possible to retrieve objects from the container by simply passing them into the parameters. Certain aspects of Masonite are resolved such as controller methods, middleware and drivers.

For example, we can hint that we want to get the `Request` class and put it into our controller. All controller methods are resolved by the container.

```python
def show(self, Request)
    Request.user()
```

In this example, Masonite will look inside the container for a key called `Request` and return that value from the container. `Request` is already loaded into the container for you out of the box.

Another way to resolve classes is by using Python 3 annotations:

```python
from masonite.request import Request

def show(self, request: Request)
    request.user()
```

Masonite will know that you are trying to get the `Request` class and will actually retrieve that class from the container. Masonite will search the container for a `Request` class regardless of what the key is in the container, retrieve it, and inject it into the controller method. Effectively creating an IOC container with dependency injection.

Pretty powerful stuff, eh?

#### Resolving your own code

The service container can also be used outside of the flow of Masonite. Masonite takes in a function or class method, and resolves it's dependencies by finding them in the service container and injecting them.

Because of this, you can resolve any of your own classes or functions.

```python
from masonite.request import Request

def randomFunction(User):
    print(User)

def show(self, Request):
    Request.app().resolve(randomFunction) # Will print the User object
```

**Remember not to call it and only reference the function. The Service Container needs to inject dependencies into the object so it requires a reference and not a callable.**

This will fetch all of the parameters of `randomFunction` and retrieve them from the service container. There probably won't be many times you'll have to resolve your own code but the option is there.