# Masonite 2.1

## Masonite 2.1

{% hint style="danger" %}
This is currently unreleased and is in a Beta 1 release. There will be 2 more beta releases in October and November until the final release of 2.1 in December.

Learn more about releases in the [Release Cycle](../prologue/release-cycle.md) documentation.
{% endhint %}

## Introduction

Masonite 2.1 introduces a few new changes that are designed to correct course for the 2.x family and ensure we can go far into the 2.x family without having to make huge breaking changes. Below you will find all changes that went into making 2.1 awesome. Nearly all of these changes are breaking changes.

{% hint style="info" %}
These are only changes in the first beta release so far so more changes may come
{% endhint %}

## All classes in core now have docstrings

It is much easier to contribute to Masonite now since nearly classes have awesome docstrings explaining what they do, their dependencies and what they return.

## Auto Resolving Parameters

We have completely removed parameter resolving. We can no longer resolve like this:

```python
def show(self, Request):
    return Request.redirect('/')
```

in favor of the more explicit:

```python
from masonite.request import Request

def show(self, request: Request):
    return request.redirect('/')
```

This is a bit of a big change and will be most of the time spent on refactoring your application to upgrade to Masonite 2.1. If you already used the more explicit version then you won't have to worry about this change. It is still possible to resolve parameters by passing that keyword argument to your container to activate that feature:

```python
container = App(resolve_parameters=True)
```

This should help with upgrading from 2.0 to 2.1 until you have refactored your application. Then you should deactivate this keyword argument so you can be in line with future 2.x releases.

You can choose to keep it activated if that is how you want to create applications but it won't be officially supported by packages, future releases or in the documentation.

## Class Middleware

All middleware are now classes:

```python
HTTP_MIDDLEWARE = [
    LoadUserMiddleware,
    CsrfMiddleware,
    ResponseMiddleware,
]
```

this is different from the previous string based middleware

```python
HTTP_MIDDLEWARE = [
    'app.http.middleware.LoadUserMiddleware.LoadUserMiddleware',
    'app.http.middleware.CsrfMiddleware.CsrfMiddleware',
    'app.http.middleware.JsonResponseMiddleware.JsonResponseMiddleware',
]
```

## Removed the Payload Input

Previously when getting an incoming JSON response, we had to get the values via the payload input like so:

```python
from masonite.request import Request

def show(self, request: Request):
    return request.input('payload')['id']
```

which was kind of strange in hindsight. Now we can just straight up use the input:

```python
from masonite.request import Request

def show(self, request: Request):
    return request.input('id')
```

Again this is only a change for incoming JSON responses. Normal form inputs remain the same.

## Removed the facades module.

Previously we had a facades module but it was being unused and we didn't see a future for this module so we moved the only class in this module to it's own class. All instances of:

```text
from masonite.facades.Auth import Auth
```

now become:

```text
from masonite.auth import Auth
```

## Provider Refactoring

### Route Provider

### Moved parameter parsing into if statement

We also noticed that for some reason we were parsing parameters before we found routes but we only ever needed those parameters inside our routes so we were parsing them whether we found a route or not. We moved the parsing of parameters into the if statement that executes when a route is found.

When we say "parsing route parameters" we mean the logic required to parse this:

```text
/dashboard/@user/@id
```

into a usable form to use on the request class this:

```python
from masonite.request import Request

def show(self, request: Request):
    request.param('user')
    request.param('id')
```

### StartResponse Provider

This provider has been completely removed for the more recommended `ResponseMiddleware` which will need to be added to your HTTP middleware list:

```python
from masonite.middleware import ResponseMiddleware
..
HTTP_MIDDLEWARE=[
    ...
    ResponseMiddleware,
]
```

We also introduced a new `Response` class which primarily handles a lot of this logic much better that is used internally for core.

## Added ability use dot notation for views

You can now optionally use `.` instead of `/` in your views:

```python
def show(self, view: View):
    return view('dashboard.user.show')
```

## Moved the CsrfMiddleware into core and extended it

We moved the CSRF middleware completely into the core framework and allow developers to extend from it now. This will allow us to fix any security bugs that are apart of the CSRF feature.

You may see this pattern a lot in the future which is only extending classes from the core framework so we can hot fix things much better.

## Completely cleaned the project

Masonite now has a plethora of docstrings on each and every class by default to really give the developer an understanding about what each default class is actually doing and what it is dependent on.

Masonite is also much more PEP 8 compliant. We removed all instances of triple single quotes: `'''` for the more preferred and PEP 8 compliant double quotes `"""` for docstrings.

We also cleaned a lot of the classes generated by the auth command since those were pretty ugly.

## Removed Helper functions by default

We also removed all instances of helper functions by default since it was confusing developers and was throwing red squiggly marks for text editors. Now all default code explicitly resolves via the container and helper functions can be used on the developers own terms.

Helper functions are still available but you will need to use them on your own terms.

## Added seeds by default

Now every application has a basic seeding structure setup which is the same as if running the `craft seed` command. This is to promote more use of this awesome feature which can be used in migration files for quick seeding of databases for development.

## Added code to \_\_init\_\_.py file in migrations and seeds

We were previously not able to import code into our migration files or database seeders because the command line tool would not pick up our current working directory to import classes into. Now the migrations module and seeds module have 3 lines of code:

```text
import os
import sys
sys.path.append(os.getcwd())
```

this helpers when running the command line to import code into these modules.

## Route Printing

In development you would see a message like:

```text
GET Route: /dashboard
```

When you hit a route in development mode. Well you would also hit it in production mode too since that was never turned off. Although this is likely fine, it would slow down the framework significantly under load since it takes a bit of resources to print something that didn't need to be printed. This enables a bit of a performance boost.

## Added Migrate:status command

You can now check the status of all your migrations by running `craft migrate:status`

## Added simple container bindings

Now you can have simple container bindings which will bind objects into the container and use the class or object name as the key:

```python
app.simple(Command1())
```

would be the same as

```python
app.bind(Command1, Command1())
```

## Removed the need to all of the safe filters

Some template helpers needed the `|safe` filter. We removed the need for that. Now:

```python
{{ csrf_field|safe }}
```

becomes:

```python
{{ csrf_field }}
```

## Added a way to set the status code using an integer

You can now set status codes like:

```python
request.status(404)
```

When previously you needed to do:

```python
request.status('404 Not Found')
```

