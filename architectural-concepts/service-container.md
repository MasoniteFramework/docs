# Service Container

## Introduction

The Service Container is an extremely powerful feature of Masonite and should be used to the fullest extent possible. It's important to understand the concepts of the Service Container. It's a simple concept but is a bit magical if you don't understand what's going on under the hood.

## Getting Started

The Service Container is just a dictionary where classes are loaded into it by key-value pairs, and then can be retrieved by either the key or value through resolving objects. That's it.

{% hint style="info" %}
Think of "resolving objects" as Masonite saying "what does your object need? Ok, I have them in this dictionary, let me get them for you."
{% endhint %}

The container holds all of the frameworks classes and features so adding features to Masonite only entails adding classes into the container to be used by the developer later on. This typically means "registering" these classes into the container \(more about this later on\).

This allows Masonite to be extremely modular.

There are a few objects that are resolved by the container by default. These include your controller methods \(which are the most common and you have probably used them so far\) driver and middleware constructors and any other classes that are specified in the documentation.

There are four methods that are important in interacting with the container: `bind`, `make` and `resolve`

## Bind

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

```python
>>> app.providers
{'User': <class app.User.User>}
```

The service container is available in the `Request` object and can be retrieved by:

```python
def show(self, Request):
    Request.app() # will return the service container
```

## **Make**

In order to retrieve a class from the service container, we can simply use the `make` method.

```python
>>> from app.User import User
>>> app.bind('User', User)
>>> app.make('User')
<class app.User.User>
```

That's it! This is useful as an IOC container which you can load a single class into the container and use that class everywhere throughout your project.

## **Resolve**

This is the most useful part of the container. It is possible to retrieve objects from the container by simply passing them into the parameters. Certain aspects of Masonite are resolved such as controller methods, middleware and drivers.

For example, we can hint that we want to get the `Request` class and put it into our controller. All controller methods are resolved by the container.

```python
def show(self, Request)
    Request.user()
```

In this example, before the show method is called, Masonite will look at the parameters and look inside the container for a key with the same name. In this example we are looking for `Request` so Masonite will look for a key inside the provider dictionary called `Request` and inject that value from the container into our method for us. `Request` is already loaded into the container for you out of the box.

Another way to resolve classes is by using Python 3 annotations:

```python
from masonite.request import Request

def show(self, request_class: Request)
    request_class.user()
```

Masonite will know that you are trying to get the `Request` class and will actually retrieve that class from the container. Masonite will search the container for a `Request` class regardless of what the key is in the container, retrieve it, and inject it into the controller method. Effectively creating an IOC container with dependency injection. Think of this as a **get by value** instead of a **get by key** like the earlier example.

{% hint style="warning" %}
Because of how objects are resolved, you can only use annotations at the end of your parameter list. For example:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
from masonite.request import Request

def show(self, request_class: Request, Upload)
    request_class.user()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

will throw an error because Masonite gets annotations last and will not catch the Upload parameter. If you need to use both normal parameters and annotations then you will need to specify them at the end:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
from masonite.request import Request

def show(self, Upload, request_class: Request)
    request_class.user()
    Upload.store(request_class.input('file'))
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endhint %}

Pretty powerful stuff, eh?

### **Resolving your own code**

The service container can also be used outside of the flow of Masonite. Masonite takes in a function or class method, and resolves it's dependencies by finding them in the service container and injecting them for you.

Because of this, you can resolve any of your own classes or functions.

```python
from masonite.request import Request

def randomFunction(User):
    print(User)

def show(self, Request):
    Request.app().resolve(randomFunction) # Will print the User object
```

{% hint style="warning" %}
Remember not to call it and only reference the function. The Service Container needs to inject dependencies into the object so it requires a reference and not a callable.
{% endhint %}

This will fetch all of the parameters of `randomFunction` and retrieve them from the service container. There probably won't be many times you'll have to resolve your own code but the option is there.

