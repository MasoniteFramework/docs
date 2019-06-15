# Masonite 2.2

## Masonite 2.2

{% hint style="danger" %}
This release is still in beta and is not yet released. All information in this documentation section is subject to change.
{% endhint %}

## Route Prefixes

Previously you had to append all routes with a `/` character. This would look something like:

```python
Get('/some/url')
```

You can now optionally prefix this without a `/` character:

```python
Get('some/url')
```

## URL parameters can now optionally be retrieved from the controller definition

Previously we had to do something like:

```python
def show(self, view: View, request: Request):
    user = User.find(request.param('user_id'))
    return view.render('some.template', {'user': user})
```

Now we can **optionally** get the parameter from the method definition:

```python
def show(self, user_id, view: View):
    user = User.find(user_id)
    return view.render('some.template', {'user': user})
```

## Added a storage manager and disk storage drivers

This is used as a wrapper around I/O operations. It will also be a wrapper around the upload drivers and moving files around and other file management type operations

## Async driver now can be specified whether to use threading or processing

We can now specify directly in the configuration file whether or not the threading or multiprocessing for the async type operations.

## Added new HTTP Verbs

We added 4 new HTTP verbs: HEAD, CONNECT, OPTIONS, TRACE. You import these and use them like normal:

```python
from masonite.routes import Connect, Trace
ROUTES = [
    Connect('..'),
    Trace('..'),
]
```

## JSON error responses

If the incoming request is a JSON request, Masonite will now return all errors as JSON

```javascript
{
  "error": {
    "exception": "Invalid response type of <class 'set'>",
    "status": 500,
    "stacktrace": [
        "/Users/joseph/Programming/core/bootstrap/start.py line 38 in app",
        "/Users/joseph/Programming/core/masonite/app.py line 149 in resolve",
        "/Users/joseph/Programming/core/masonite/providers/RouteProvider.py line 92 in boot",
        "/Users/joseph/Programming/core/masonite/response.py line 105 in view"
    ]
  }
}
```

## Rearranged Drivers into their own folders

This is more of an internal change for Core itself.

## Craft serve command defaults to auto-reloading

Before we had to specify that we wanted the server to auto-reload by specifying a -r flag:

```bash
$ craft serve -r
```

Now we can just specify the serve command it will default to auto-reloading:

```bash
$ craft serve
```

You can now specify it to NOT auto-reload by passing in 1 of these 2 commands:

```text
$ craft serve -d
$ craft serve --dont-reload
```

## Added Accept\(\*\) to drivers

By default you can only upload image files because of security reasons but now you can disable that by doing an accept\(\*\) option:

```python
def show(self, upload: Upload):
    upload.accept(*).store(request.input('file'))
```

## Added much more view helpers

A list of view helpers can be [found here](../the-basics/views.md#helpers)

## All Tests are now unittests

We moved from pytest to unittests for test structures.

## Added a better way to run database tests

Added a new `DatabaseTestcase` so we can properly setup and teardown our database. This works for sqlite databases by default to prevent your actual database from being destroyed.

## The back view helper now defaults to the current path

Before in templates we had to specify a path to go back to but most of the time we wanted to go back to the current path. 

Instead of:

```markup
<form ..>
    {{ back(request().path) }}
</form>
```

We can now do:

```markup
<form ..>
    {{ back() }}
</form>
```

In order to learn how to use this you can visit the [documentation here](../the-basics/requests.md#form-back-redirection).

## Added a completely new validation library

We built a new validation library from scratch and completely ripped out the old validation code. Any current validation code will need to be updated to the new way.

The new way is MUCH better. You can read about it in the new [validation section here](https://docs.masoniteproject.com/v/v2.2/advanced/validation).

## Auth class does not need the request class.

Previously we needed to pass in the request object to the Auth class like this:

```python
from masonite.auth import Auth
from masonite.request import Request

def show(self, request: Request):
    Auth(request).login(..)    
```

Now we have it a bit cleaner and you can just resolve it and the request class will be injected for you

```python
from masonite.auth import Auth
from masonite.request import Request

def show(self, request: Request, auth: Auth):
    auth.login(..)  
```

## Completely changed how classes are resolved on the backend

You may not notice anything but now if you bind a class into the container like this:

```python
from masonite.auth import Auth

def register(self):
    self.app.bind('Auth', Auth)
```

It will be resolved when you resolve it:

```python
from masonite.auth import Auth

def show(self, auth: Auth):
    auth.login(..)
```

This is why the Auth class no longer needs to accept the request class. Masonite will inject the request class for you when you resolve the class. 

This works with all classes and even your custom classes to help manage your application dependencies

## Added a new register method for authentication

In order for Masonite to be more pluggable and modular, we stopped hardcoding how registering was done so we can add new drivers in the future.

{% hint style="info" %}
Read the [authentication documentation](../security/authentication.md) for more information.
{% endhint %}

## Moved all regex compiling before the server boots

There were a lot of needless computations behind done to constantly recompile regex that has already been compiled before. Now all routes are responsible for compiling their own regex **when they are constructed**. This offsets a lot of computations before the server even boots. This is a huge performance boost.

## Added Container Remembering

The container can now remember previous objects it has already resolved. This can lead to a performance boost of 10 - 15x when it comes to container resolving.

## Added with\_errors to the request class

Previous we had to flash errors to the session and then redirect back. Now we can do both at the same time.

This takes this code example:

```python
def show(self, request: Request):
    errors = request.validate(
        required('field')
    )
    
    if errors:
        request.session.flash('errors', errors)
        request.back()
```

Now we can do this:

```python
def show(self, request: Request):
    errors = request.validate(
        required('field')
    )
    
    if errors:
        request.back().with_errors(errors)
```

## Added the concept of publishing

In order to assist in package development, it is now easier to publish assets like migrations, routes, and commands from your package and into the developers application

## Added a new JWT driver for authentication

Previously every request required a database call. Now you can set the driver to `jwt` and it will store all the user information into a `jwt` token, encrypted as a cookie and continuously fetch the information from the token instead of calling the database on every request.

