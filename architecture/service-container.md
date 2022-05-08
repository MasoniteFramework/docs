The Service Container is an extremely powerful feature of Masonite and should be used to the fullest extent possible. It's important to understand the concepts of the Service Container. It's a simple concept but is a bit magical if you don't understand what's going on under the hood.

### Getting Started

The Service Container is just a dictionary where classes are loaded into it by key-value pairs, and then can be retrieved by either the key or value through resolving objects. That's it.

{% hint style="info" %}
Think of "resolving objects" as Masonite saying "what does your object need? Ok, I have them in this dictionary, let me get them for you."
{% endhint %}

The container holds all of the frameworks classes and features so adding features to Masonite only entails adding classes into the container to be used by the developer later on. This typically means "registering" these classes into the container \(more about this later on\).

This allows Masonite to be extremely modular.

There are a few objects that are resolved by the container by default. These include your controller methods \(which are the most common and you have probably used them so far\) driver and middleware constructors and any other classes that are specified in the documentation.

There are three methods that are important in interacting with the container: `bind`, `make` and `resolve`

### Bind

In order to bind classes into the container, we will just need to use a simple `bind` method on our `app` container. In a service provider, that will look like:

```python
from masonite.provider import ServiceProvider
from app.User import User

class UserModelProvider(ServiceProvider):

    def register(self):
        self.applicationlication.bind('User', User)

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
def show(self, request: Request):
    request.app() # will return the service container
```

### Simple Binding

Sometimes you really don't care what the key is for the object you are binding. For example you may be binding a `Markdown` class into the container but really don't care what the key binding is called. This is a great reason to use simple binding which will set the key as the object class:

```python
from masonite.provider import ServiceProvider
from app.User import User


class UserModelProvider(ServiceProvider):

    def register(self):
        self.application.simple(User)

    def boot(self):
        pass
```

### Make

In order to retrieve a class from the service container, we can simply use the `make` method.

```python
>>> from app.User import User
>>> app.bind('User', User)
>>> app.make('User')
<class app.User.User>
```

That's it! This is useful as an IOC container which you can load a single class into the container and use that class everywhere throughout your project.

### Singleton

You can bind singletons into the container. This will resolve the object **at the time of binding**. This will allow the same object to be used throughout the lifetime of the server.

```python
from masonite.provider import ServiceProvider
from app.helpers import SomeClass


class UserModelProvider(ServiceProvider):

    def register(self):
        self.application.singleton('SomeClass', SomeClass)

    def boot(self):
        pass
```

### Has

You can also check if a key exists in the container by using the `has` method:

```python
app.has('request')
```

You can also check if a key exists in the container by using the `in` keyword.

```python
'request' in app
```

### Collecting

You may want to collect specific kinds of objects from the container based on the key. For example we may want all objects that start with "Exception" and end with "Hook" or want all keys that end with "ExceptionHook" if we are building an exception handler.

### Collect By Key

We can easily collect all objects based on a key:

```python
app.collect('*ExceptionHook')
```

This will return a dictionary of all objects that are binded to the container that start with anything and end with "ExceptionHook" such as "SentryExceptionHook" or "AwesomeExceptionHook".

We can also do the opposite and collect everything that starts with a specific key:

```python
app.collect('Sentry*')
```

This will collect all keys that start with "Sentry" such as "SentryWebhook" or "SentryExceptionHandler."

Lastly, we may want to collect things that start with "Sentry" and end with "Hook"

```python
app.collect('Sentry*Hook')
```

This will get keys like "SentryExceptionHook" and "SentryHandlerHook"

### Collecting By Object

You can also collect all subclasses of an object. You may use this if you want to collect all instances of a specific class from the container:

```python
from cleo import Command
...
app.collect(Command)
# Returns {'FirstCommand': <class ...>, 'AnotherCommand': ...}
```

### Resolve

This is the most useful part of the container. It is possible to retrieve objects from the container by simply passing them into the parameter list of any object. Certain aspects of Masonite are resolved such as controller methods, middleware and drivers.

For example, we can type hint that we want to get the `Request` class and put it into our controller. All controller methods are resolved by the container.

```python
def show(self, request: Request):
    request.user()
```

In this example, before the show method is called, Masonite will look at the parameters and look inside the container for the Request object.

Masonite will know that you are trying to get the `Request` class and will actually retrieve that class from the container. Masonite will search the container for a `Request` class regardless of what the key is in the container, retrieve it, and inject it into the controller method. Effectively creating an IOC container with dependency injection. capabilities Think of this as a **get by value** instead of a **get by key** like the earlier example.

Pretty powerful stuff, eh?

Masonite will also resolve your custom, application-specific classes **including those that you have not explicitly bound with `app.bind()`.**

Continuing with the example above, the following will work out of the box (assuming the relevant classes exist), without needing to bind the custom classes in the container:

```python
# elsewhere...
class MyService:
    def __init__(self, some_other_dependency: SomeOtherClass):
        pass
    
    def do_something(self):
        pass


# in a controller...
def show(self, request: Request, service: MyService):
    request.user()
    service.do_something()
```


### Resolving Instances

Another powerful feature of the container is it can actually return instances of classes you annotate. For example, all `Upload` drivers inherit from the `UploadContract` which simply acts as an interface for all `Upload` drivers. Many programming paradigms say that developers should code to an interface instead of an implementation so Masonite allows instances of classes to be returned for this specific use case.

Take this example:

```python
from masonite.contracts import UploadContract

def show(self, upload: UploadContract)
    upload # <class masonite.drivers.UploadDiskDriver>
```

Notice that we passed in a contract instead of the upload class. Masonite went into the container and fetched a class with the instance of the contract.

### Resolving your own code

The service container can also be used outside of the flow of Masonite. Masonite takes in a function or class method, and resolves it's dependencies by finding them in the service container and injecting them for you.

Because of this, you can resolve any of your own classes or functions.

```python
from masonite.request import Request
from masonite.view import View

def randomFunction(view: View):
    print(view)

def show(self, request: Request):
    request.app().resolve(randomFunction) # Will print the View object
```

{% hint style="warning" %}
Remember not to call it and only reference the function. The Service Container needs to inject dependencies into the object so it requires a reference and not a callable.
{% endhint %}

This will fetch all of the parameters of `randomFunction` and retrieve them from the service container. There probably won't be many times you'll have to resolve your own code but the option is there.

### Resolving With Additional Parameters

Sometimes you may wish to resolve your code in addition to passing in variables within the same parameter list. For example you may want to have 3 parameters like this:

```python
from masonite.request import Request
from masonite import Mail

def send_email(request: Request, mail: Mail, email):
    pass
```

You can resolve and pass parameter at the same time by adding them to the `resolve()` method:

```python
app.resolve(send_email, 'user@email.com')
```

Masonite will go through each parameter list and resolve them, if it does not find the parameter it will pull it from the other parameters specified. These parameters can be in any order.

### Using the container outside of Masonite flow

If you need to utilize a container outside the normal flow of Masonite like inside a command then you can import the container directly.

This would look something like:

```python
from wsgi import container
from masonite import Queue

class SomeCommand:

    def handle(self):
        queue = container.make(Queue)
        queue.push(..)
```

### Container Swapping

Sometimes when you resolve an object or class, you want a different value to be returned.

### Using a value:

We can pass a simple value as the second parameter to the `swap` method which will be returned instead of the object being resolved. For example this is used currently when resolving the `Mail` class like this:

```python
from masonite import Mail

def show(self, mail: Mail):
    mail #== <masonite.drivers.MailSmtpDriver>
```

but the class definition for the `Mail` class here looks like this:

```python
class Mail:
    pass
```

How does it know to resolve the smtp driver instead? It's because we added a container swap. Container swaps are simple, they take the object as the first parameter and either a value or a callable as the second.

For example we may want to mock the functionality above by doing something like this in the boot method of a Service Provider:

```python
from masonite import Mail

def boot(self, mail: MailManager):
    self.application.swap(Mail, manager.driver(self.application.make('MailConfig').DRIVER))
```

Notice that we specified which class should be returned whenever we resolve the `Mail` class. In this case we want to resolve the default driver specified in the projects configurations.

### Using a callable

Instead of directly passing in a value as the second parameter we can pass in a callable instead. The callable MUST take 2 parameters. The first parameter will be the annotation we are trying to resolve and the second will be the container itself. Here is an example of how the above would work with a callable:

```python
from masonite import Mail
from somewhere import NewObject
...

def mail_callback(obj, container):
    return NewObject
...

def boot(self):
    self.application.swap(Mail, mail_callback)
```

Notice that the second parameter is a callable object. This means that it will be called whenever we try to resolve the `Mail` class.

Remember: If the second parameter is a callable, it will be called. If it is a value, it will simply be returned instead of the resolving object.

### Container Hooks

Sometimes we might want to run some code when things happen inside our container. For example we might want to run some arbitrary function about we resolve the Request object from the container or we might want to bind some values to a View class anytime we bind a Response to the container. This is excellent for testing purposes if we want to bind a user object to the request whenever it is resolved.

We have three options: `on_bind`, `on_make`, `on_resolve`. All we need for the first option is the key or object we want to bind the hook to, and the second option will be a function that takes two arguments. The first argument is the object in question and the second argument is the whole container.

The code might look something like this:

```python
from masonite.request import Request

def attribute_on_make(request_obj, container):
    request_obj.attribute = 'some value'

...

container = App()

# sets the hook
container.on_make('request', attribute_on_make)
container.bind('request', Request)

# runs the attribute_on_make function
request = container.make('request')
request.attribute # 'some value'
```

Notice that we create a function that accepts two values, the object we are dealing with and the container. Then whenever we run `on_make`, the function is ran.

We can also bind to specific objects instead of keys:

```python
from masonite.request import Request

# ...

# sets the hook
container.on_make(Request, attribute_on_make)
container.bind('request', Request)

# runs the attribute_on_make function
request = container.make('request')
request.attribute # 'some value'
```

This then calls the same attribute but anytime the `Request` object itself is made from the container. Notice everything is the same except line 6 where we are using an object instead of a string.

We can do the same thing with the other options:

```python
container.on_bind(Request, attribute_on_make)
container.on_make(Request, attribute_on_make)
container.on_resolve(Request, attribute_on_make)
```


