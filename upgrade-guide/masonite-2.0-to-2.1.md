# Masonite 2.0 to 2.1

## Masonite 2.0 to 2.1

## Introduction

Masonite 2.1 is a fantastic release. It works out a lot of the kinks that were in 2.0 as well as brings several new syntactically good looking code generation

This guide just shows the major changes between version to get your application working on 2.1. You should see the [What's New in 2.1](../whats-new/masonite-2.1.md) documentation to upgrade smaller parts of your code that are likely to be smaller quality of life improvements.

### Masonite CLI

For 2.1 you will need `masonite-cli>=2.1.0`.

Make sure you run:

```text
$ pip install masonite-cli --upgrade
```

### Middleware

Middleware has been changed to classes so instead of doing this in your `config/middleware.py` file:

```python
HTTP_MIDDLEWARE = [
    'app.http.middleware.DashboardMiddleware.DashboardMiddleware',
]
```

You will now import it directly:

```python
import app.http.middleware.DashboardMiddleware import DashboardMiddleware

HTTP_MIDDLEWARE = [
    DashboardMiddleware,
]
```

### Auto resolving parameters has been removed

This is likely the biggest change in 2.0. Before 2.1 you were able to fetch by key when resolving by doing something like:

```python
def show(self, Request):
    Request.input(..)
```

We have removed this by default and now you much explicitly import your classes in order to interact with the container resolving:

```python
from masonite.request import Request

def show(self, request: Request):
    request.input(..)
```

If you truly do not like this change you can modify your container on a per project basis by adding this to your container constructor in `wsgi.py`:

```python
container = App(resolve_parameters=True)
```

Just know this is not recommended and Masonite may or may not remove this feature entirely at some point in the future.

### Resolving Mail, Queues and Broadcasts

Previously we were able to do something like this:

```python
def show(self, Mail):
    Mail.to(..)
```

Since we never actually created a class from this and you were not able to explicitly resolve this, we utilized the new container swapping in order to swap a class out for this container binding.

All instances above should be changed to:

```python
from masonite import Mail

def show(self, mail: Mail):
    mail.to(..)
```

Don't forgot to also do your `boot` methods on your Service Providers as well:

```python
def boot(self, request: Request):
    ..request..
```

As well as all your middleware and custom code:

```python
class AuthenticationMiddleware(object):
    """ Middleware To Check If The User Is Logged In """

    def __init__(self, request: Request):
        """ Inject Any Dependencies From The Service Container """
        self.request = request
    ...
```

### Resolving your own code

You may have classes you binded personally to the container like this:

```python
def slack_send(self, IntegrationManager):
    return IntegrationManager.driver('slack').scopes('incoming-webhook').state(self.request.param('id')).redirect()
```

To get this in line for 2.1 you will need to use Container Swapping in order to be able to resolve this. This is actually an awesome feature.

First go to your provider where you binded it to the container:

```python
from app.managers import IntegrationManager

def boot(self):
    self.app.bind('IntegrationManager', IntegrationManager.driver('something'))
```

and add a container swap right below it by swapping it with a class:

```python
from app.managers import IntegrationManager

def boot(self):
    self.app.bind('IntegrationManager', IntegrationManager.driver('somedriver'))

    self.app.swap(IntegrationManager, IntegrationManager.driver('somedriver'))
```

now you can use that class to resolve:

```python
from app.managers import IntegrationManager

def slack_send(self, manager: IntegrationManager):
    return manager.driver('slack').scopes('incoming-webhook').state(self.request.param('id')).redirect()
```

### Removed Masonite Facades

Completely removed the `masonite.facades` module and put the only class \(the `Auth` class\) in the `masonite.auth` module.

So all instances of:

```python
from masonite.facades.Auth import Auth
```

need to be changed to:

```python
from masonite.auth import Auth
```

### Removed the StartResponseProvider

The `StartResponseProvider` was not doing anything crazy and it could be achieved with a simple middleware. This speeds up Masonite slightly by offsetting where the response preparing takes place.

Simply remove the `StartResponseProvider` from your `PROVIDERS` list:

```python
PROVIDERS = [
    # Framework Providers
    AppProvider,
    SessionProvider,
    RouteProvider,
    StatusCodeProvider,
    # StartResponseProvider,
    WhitenoiseProvider,
    ViewProvider,
    HelpersProvider,

    ...
```

As well as put the new middleware in the HTTP middleware

```python
from masonite.middleware import ResponseMiddleware
..

HTTP_MIDDLEWARE = [
    LoadUserMiddleware,
    CsrfMiddleware,
    HtmlMinifyMiddleware,
    ResponseMiddleware, # Here
]
```

### JSON Payloads

In 2.0 you had to fetch incoming JSON payloads like this:

```python
request.input('payload')['id']
```

So now all instances of the above can be used normally:

```python
request.input('id')
```

### Moved CSRF Middleware into core

CSRF middleware now lives in core and allows you to override some methods or interact with the middleware with class attributes:

Replace your current CSRF Middleware with this new one:

```python
""" CSRF Middleware """

from masonite.middleware import CsrfMiddleware as Middleware


class CsrfMiddleware(Middleware):
    """ Verify CSRF Token Middleware """

    exempt = []
```

If you made changes to the middleware to prevent middleware from being ran on every request you can now set that as a class attribute:

```python
""" CSRF Middleware """

from masonite.middleware import CsrfMiddleware as Middleware


class CsrfMiddleware(Middleware):
    """ Verify CSRF Token Middleware """

    exempt = []
    every_request = False
```

This also allows any security issues found with CSRF to be handled on all projects quickly instead of everyone having to patch their applications individually.

### Added cwd imports to migrations and seeds

In migrations \(and seeds\) you will need to put this import inside a `__init__.py` file in order to allow models to be imported into them

```python
import os
import sys
sys.path.append(os.getcwd())
```

### Bootstrap File

There was a slight change in the `bootstrap/start.py file` around `line 60`.

This line:

```python
 start_response(container.make('StatusCode'), container.make('Headers'))
```

Needs to be changed to:

```python
start_response(
    container.make('Request').get_status_code(), 
    container.make('Request').get_and_reset_headers()
)
```

### Response Binding

You no longer should bind directly to the `Response` key in the container. You should use the new Response object.

All instances of:

```python
self.app.bind('Response', 'some value')
```

should now be:

```python
response.view('some value')
```

and any instance of:

```text
self.app.make('Response')
```

should be changed to:

```text
response.data()
```

Restructuring some sample code would be changing this:

```python
self.request.app().bind(
    'Response',
    htmlmin.minify(
        self.request.app().make('Response')
    )
)
```

to this:

```python
self.response.view(
    htmlmin.minify(self.response.data())
)
```

### Cache Exists name change

The `Cache.cache_exists()` has been changed to just `Cache.exists()`. You will need to make changes accordingly:

```python
from masonite import Cache

def show(self, cache: Cache):

    # From
    cache.cache_exists('key')

    # To
    cache.exists('key')
```

That is all the main changes in 2.1. Go ahead and run your server and you should be good to go. For a more up to date list on small improvements that you can make in your application be sure to checkout the [Whats New in 2.1](../whats-new/masonite-2.1.md) documentation article.

## Environment Variables

Although not a critical upgrade, it would be a good idea to replace all instances of retrieval of environment variables with the new `masonite.env` function.

Change all instances of this:

```python
import os
..

DRIVER = os.getenv('key', 'default')
..
KEY = os.envrion.get('key', 'default')
```

with the new `env` function:

```python
from masonite import env
..

DRIVER = env('key', 'default')
..
KEY = env('key', 'default')
```

What this will do is actually type cast accordingly. If you pass a numeric value it will cast it to an int and if you want a boolean if will cast `True`, `true`, `False`, `false` to booleans like this:

if you don't want to cast the value you can set the `cast` parameter to `False`

```python
KEY = env('key', 'default', cast=False)
```

## Removed Store Prepend method

We removed the `store_prepend()` method on the upload drivers for the `filename` keyword arg on the store method.

So this:

```text
upload.store_prepend('random-string', request.input('file'))
```

now becomes:

```text
upload.store(request.input('file'), filename='random-string')
```

