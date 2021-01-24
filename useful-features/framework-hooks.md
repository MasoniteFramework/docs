# Framework Hooks

## Introduction

Framework hooks are essentially events that are emitted that you are able to "hook" into. If you want to add support for [Sentry](http://sentry.io), which is an exception and error tracking solution, then you want to tie into the Exception Hook. When an exception is encountered it will look for your class in the container and execute it.

{% hint style="info" %}
Currently there is only the Exception Hook that you can tie into but there will be other hooks in later releases of Masonite such as View Hooks and Mail Hooks.
{% endhint %}

## Getting Started

For the purposes of learning about Framework Hooks, we will walk through how to implement [Sentry](http://sentry.io) into a Masonite application by adding it to the Exception Hook.

## Exception Hooks

The Exception Hook is fired when an exception is thrown in an application. Anytime you would normally see the debug view when developing is when this hook will fire. This hook may not be fired if the server does not boot up because of an exception depending on how far into the container the exception is thrown but any normal application exceptions encountered will fire this hook.

The exception hook to tie into is ExceptionHook. This means that the key in the container should end with ExceptionHook and Masonite will call it when the Exception Hook is thrown. We can load things into the container such as:

```python
app.bind('SentryExceptionHook', YourObject())
```

Notice here that our key ends with ExceptionHook and that we instantiated the object. Let's explore creating this entirly from scratch.

### Creating A Hook

Let's explore how we can simply add [Sentry](http://sentry.io) to our application. This should take about 5 minutes.

Let's create a class called `SentryHook` and put it into `app/hooks/sentry.py`.

{% code title="app/hooks/sentry.py" %}
```python
class SentryHook:
    def __init__(self):
        pass

    def load(self, app):
        self._app = app
```
{% endcode %}

This should be the basic structure for a hook. All hooks require a load method. This load method will always be passed the application container so it always requires that parameter. From here we can do whatever we need to by making objects from the container.

But for this example we actually don't need the container so we can ignore it.

#### Adding Sentry

Ok great so now let's add sentry to our application. You can sign up with [Sentry.io](http://sentry.io) which will show you the basic 3 lines you need to add Sentry to any Python project.

This should be the finished hook:

```python
from raven import Client
client = Client( 'https://874..:j8d@sentry.io/1234567')

class SentryHook:
    def __init__(self):
        pass

    def load(self, app):
        client.captureException()
```

That's it. The key in the client should be available through the [Sentry.io](http://sentry.io) dashboard.

### Tieing Into The Exception Hook

Now let's walk through how we can simply tie this into the container so it will be called when an exception is thrown in our project.

{% hint style="info" %}
Remeber that all we need to do is call is add it to the container and append the correct string to the key.
{% endhint %}

We can create a new Service Provider to store our hooks so let's make one.

```text
$ python craft provider SentryServiceProvider
```

This will create a new Service Provider inside `app/providers/SentryServiceProvider.py` that looks like:

```python
from masonite.provider import ServiceProvider

class SentryServiceProvider(ServiceProvider):
    def register(self): 
        pass

    def boot(self): 
        pass
```

Now let's just add our hook to it:

```python
from masonite.provider import ServiceProvider
from ..hooks.sentry import SentryHook

class SentryServiceProvider(ServiceProvider):
    def register(self): 
        self.app.bind('SentryExceptionHook', SentryHook())

    def boot(self): 
        pass
```

{% hint style="info" %}
Notice that the key we binded our object to ends with "ExceptionHook." What we put before this part of the string is whatever you want to put. Also note that we also instantiated our `SentryHook()` and didn't put `SentryHook`
{% endhint %}

And finally add the Service Provider to our `PROVIDERS` constant in our `config/providers.py` file:

```python
from app.providers.SentryServiceProvider import SentryServiceProvider
...
PROVIDERS = [
    # Framework Providers
    AppProvider,
...
    ViewProvider,

    # Optional Framework Providers
    SassProvider,
    MailProvider,
...
    # Application Providers
    SentryServiceProvider
...
]
...
```

That's it! Now everytime an exception is thrown, this will run our SentryHook class that we binded into the container and the exception should pop up in our [Sentry.io](http://sentry.io) dashboard.

## Exception Handlers

You can build handlers to handle specific exceptions that are thrown by your application. For example if a TemplateNotFound exception is thrown then you can build a special exception handler to catch that and return a special view or a special debug screen.

### Building an Exception Handler

Exception handlers are simple classes that have a handle method which accepts the exception thrown:

```python
from masonite.request import Request

class TemplateNotFoundHandler:

    def __init__(self, request: Request):
        self.request = request

    def handle(self, exception):
        pass
```

The constructor of all exception handlers are resolved by the container so you can hint any dependencies you need. Once the constructor is resolved then the handle method will be called.

Once the handle method is called then Masonite will continue with the rest of the WSGI logic which is just setting the status code, headers and returning a response.

### Registering the Exception Handler

Now that we have our exception handler we will need to register it in the container using a special naming convention. The naming convention is: `ExceptionNameOfErrorHandler`. The name of the exception that we want to be catching is called TemplateNotFound so we will need to bind this into the container like so:

```python
from masonite.provider import ServiceProvider
from somewhere import TemplateNotFoundHandler

class UserModelProvider(ServiceProvider):

    def register(self): 
        self.app.bind('ExceptionTemplateNotFoundHandler', TemplateNotFoundHandler)

    ...
```

## Exception Listeners

While an exception handler will actually handle the incoming exception, exception listeners are a little different.

Masonite can have several listeners registered with the framework that will listen to specific \(or all\) exceptions and have that exception passed into it if one is raised. It will then perform any logic it needs until an exception handler finally handles the exception.

### Creating a Listener

A listener is a simple class that requires a list of exceptions to listen for. Here is a simple boilerplate of an exception listener:

```python
from masonite.request import Request
from masonite.listeners import BaseExceptionListener

class ExceptionListener(BaseExceptionListener):

    listens = [
        ZeroDivisionError
    ]

    def __init__(self, request: Request):
        self.request = request

    def handle(self, exception, file, line):
        # Perform an action
```

Note that the `__init__` method of a listener is resolved by the container so feel free to type hint anything you need there.

Finally it requires a `handle` method which takes 3 arguments. the `exception` that was thrown, the `file` that the exception was thrown in and the `line` the exception was thrown on.

You can either listen to any number of exceptions or all exceptions by passing in a `*` to the `listens` attribute:

```python
class ExceptionListener(BaseExceptionListener):

    listens = ['*']

    # ...
```

A good use case for this would be Masonite Logging package which uses this to log any exceptions.

### Registering Exception Listeners

You can register an exception listener directly to the container with any service provider. For easy use, you can use a `simple` bind to the container which will bind the class to the container with the name of the class as the key:

```python
from some.place import LoggerExceptionListener

class YourProvider:

    wsgi = False

    def register(self):
        self.app.simple(LoggerExceptionListener)
```

Your listener will now run whenever an exception occurs that your listener is listening to.

