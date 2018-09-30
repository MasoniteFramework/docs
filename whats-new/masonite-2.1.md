# Masonite 2.1

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
def show(self, request: Request):
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
    JsonResponseMiddleware
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

## RouteProvider Refactoring

### JsonResponseMiddleware

We refactored a lot of the ResponseProvider which is the provider with the most complex logic to make the framework work and is responsible for all the logic involved in finding and parsing route and controller logic.

In this refactoring we moved a few things out and abstracted them away. For one we moved the parsing of JSON responses to it's own JsonResponseMiddleware.

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

## Added ability to set your template splices

Currently we splice our templates using the `/` character like this:

```python
return view('template/some/directory')
```

but we can set that character in a boot method of a Service Provider:

```python
from masonite.view import View

def boot(self, view: View):
    view.set_template_splice('.')
```

and then use it like this in all of our controllers:

```python
return view('template.some.directory')
```

It's a little bit cleaner and we can do it on a per project basis.

## Moved the CsrfMiddleware into core and extended it

We moved the CSRF middleware completely into the core framework and allow developers to extend from it now. This will allow us to fix any security bugs that are apart of the CSRF feature.

You may see this pattern a lot in the future which is only extending classes from the core framework so we can hot fix things much better.

## Completely cleaned the project

Masonite now has a plethora of docstrings on each and every class by default to really give the developer an understanding about what each default class is actually doing and what it is dependent on.

Masonite is also much more PEP 8 compliant. We removed all instances of triple single quotes: `'''` for the more preferred and PEP 8 compliant double quotes `"""` for docstrings.

We also cleaned a lot of the classes generated by the auth command since those were pretty ugly.

## Removed Helper functions by default

We also removed all instances of helper functions by default since it was confusing developers and was throwing red squiggly marks for text editors. Now all default code explicitly resolves via the container and helper functions can be used on the developers own terms.

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

