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

{% hint style="success" %}
Learn more in the [Controllers documentation here](../the-basics/controllers.md#passing-route-parameters).
{% endhint %}

## Added a storage manager and disk storage drivers

This is used as a wrapper around I/O operations. It will also be a wrapper around the upload drivers and moving files around and other file management type operations

## Async driver now can be specified whether to use threading or processing

We can now specify directly in the configuration file whether or not the threading or multiprocessing for the async type operations.

{% hint style="success" %}
Learn more in the [Queues documentation here](../useful-features/queues-and-jobs.md#async-driver).
{% endhint %}

## Added new HTTP Verbs

We added 4 new HTTP verbs: `HEAD`, `CONNECT`, `OPTIONS`, `TRACE`. You import these and use them like normal:

```python
from masonite.routes import Connect, Trace
ROUTES = [
    Connect('..'),
    Trace('..'),
]
```

{% hint style="success" %}
Learn more in the [Routes documentation here](../the-basics/routing.md#http-verbs).
{% endhint %}

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

{% hint style="success" %}
Learn more in the [Craft commands documentation here](../the-craft-command/introduction.md#running-the-wsgi-server).
{% endhint %}

## Added Accept\('\*'\) to drivers

By default you can only upload image files because of security reasons but now you can disable that by doing an accept\('\*'\) option:

```python
def show(self, upload: Upload):
    upload.accept('*').store(request.input('file'))
```

{% hint style="success" %}
Learn more in the [Uploading documentation here](../useful-features/uploading.md#accepting-specific-files).
{% endhint %}

## Added much more view helpers

{% hint style="success" %}
Learn more in the [Views documentation here](../the-basics/views.md#helpers).
{% endhint %}

## All Tests are now unittests

We moved from pytest to unittests for test structures.

{% hint style="success" %}
Learn more in the [Testing documentation here](../useful-features/testing.md#testing).
{% endhint %}

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

{% hint style="success" %}
Learn more in the [Requests documentation here](../the-basics/requests.md#redirecting-back).
{% endhint %}

## Added a completely new validation library

We built a new validation library from scratch and completely ripped out the old validation code. Any current validation code will need to be updated to the new way.

The new way is MUCH better. You can read about it in the new [validation section here](https://docs.masoniteproject.com/v/v2.2/advanced/validation).

{% hint style="success" %}
Learn more in the [Validation documentation here](../advanced/validation.md#validation).
{% endhint %}

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

## Added new register method to the `Auth` class.

You can now do something like:

```python
from masonite.auth import Auth

def show(self, auth: Auth):
    auth.register({
        'name': 'Joe',
        'email': 'joe@email.com',
        'password': 'secret'
    })
```

{% hint style="success" %}
Learn more in the [Authentication documentation here](../security/authentication.md#authentication).
{% endhint %}

## Changed all regex compiling to be done before the server starts

Previously, each route's regex was being compiled when Masonite checked for it but we realized this was redundant. So now all route compiling is done before the server starts.

This has given Masonite a bit of a speed boost.

## Container Remembering

Masonite now has the ability to remember the previous container bindings for each object. This can speed of resolving your code by 10-15x. This is disabled by default as it is still not clear what kind of issues this can cause.

This is scheduled to be set by default in the next major version of Masonite

{% hint style="success" %}
Learn more in the [Service Container documentation here](../architectural-concepts/service-container.md#remembering).
{% endhint %}

## Added a new `with_errors()` method in order to cut down on setting an errors session.

Now instead of doing this:

```python
from masonite.request import Request
from masonite.validation import Validator

def show(self, request: Request, validate: Validator):
    errors = request.validate(
      validate.required('user')
    )

    if errors:
      request.session.flash('errors', errors)
      return request.back()
```

we can now shorten down the flashing of errors and do:

```python
from masonite.request import Request
from masonite.validation import Validator

def show(self, request: Request, validate: Validator):
    errors = request.validate(
      validate.required('user')
    )

    if errors:
      return request.back().with_errors(errors)
```

{% hint style="success" %}
Learn more in the [Requests documentation here](../advanced/validation.md#validating-the-request).
{% endhint %}

