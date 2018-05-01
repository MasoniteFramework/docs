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

{% code-tabs %}
{% code-tabs-item title="app/hooks/sentry.py" %}
```python
class SentryHook:
    def __init__(self):
        pass
    
    def load(self, app):
        self._app = app
```
{% endcode-tabs-item %}
{% endcode-tabs %}

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
$ craft provider HookProvider
```

This will create a new Service Provider inside `app/providers/HookProvider.py` that looks like:

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

And finally add the Service Provider to our `PROVIDERS` constant in our `config/application.py` file:

```python
...
# Application Providers 
'app.providers.UserModelProvider.UserModelProvider',
'app.providers.MiddlewareProvider.MiddlewareProvider',

# Sentry Provider
'app.providers.SentryServiceProvider.SentryServiceProvider',
...
```

That's it! Now everytime an exception is thrown, this will run our SentryHook class that we binded into the container and the exception should pop up in our [Sentry.io](http://sentry.io) dashboard.





