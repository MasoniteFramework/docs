# Masonite 2.1

### Introduction

Masonite 2.1 introduces a few new changes that are designed to correct course for the 2.x family and ensure we can go far into the 2.x family without having to make huge breaking changes. It was questionable whether we should break from the 2.x family and start a new 3.x line. The biggest question was removing \(actually disabling\) the ability to resolve parameters and go with the more favorable annotation resolving. That could have made Masonite a 3.x line but we have ultimately decided to go with the 2.1 as a course correction. Below you will find all changes that went into making 2.1 awesome. Nearly all of these changes are breaking changes.xw

### All classes in core now have docstrings

It is much easier to contribute to Masonite now since nearly classes have awesome docstrings explaining what they do, their dependencies and what they return.

### Auto Resolving Parameters

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

### Class Middleware

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

### Removed the Payload Input

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

### Removed the facades module.

Previously we had a facades module but it was being unused and we didn't see a future for this module so we moved the only class in this module to it's own class. All instances of:

```text
from masonite.facades.Auth import Auth
```

now become:

```text
from masonite.auth import Auth
```

## Provider Refactoring

## Route Provider

#### Moved parameter parsing into if statement

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

This provider has been completely removed for the more recommended ResponseMiddleware which will need to be added to your HTTP middleware list:

```python
from masonite.middleware import ResponseMiddleware
..
HTTP_MIDDLEWARE=[
    ...
    ResponseMiddleware,
]
```

#### Moved parameter parsing into if statement

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

## StartResponse Provider

This provider has been completely removed for the more recommended ResponseMiddleware which will need to be added to your HTTP middleware list:

```python
from masonite.middleware import ResponseMiddleware
..
HTTP_MIDDLEWARE=[
    ...
    ResponseMiddleware,
]
```

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

## Added ability use dot notation for views

You can now optionally use `.` instead of `/` in your views:

```python
def show(self, view: View):
    return view.render('dashboard.user.show')
```

### Moved the CsrfMiddleware into core and extended it

We moved the CSRF middleware completely into the core framework and allow developers to extend from it now. This will allow us to fix any security bugs that are apart of the CSRF feature.

You may see this pattern a lot in the future which is only extending classes from the core framework so we can hot fix things much better.

### Completely cleaned the project

Masonite now has a plethora of docstrings on each and every class by default to really give the developer an understanding about what each default class is actually doing and what it is dependent on.

Masonite is also much more PEP 8 compliant. We removed all instances of triple single quotes: `'''` for the more preferred and PEP 8 compliant double quotes `"""` for docstrings.

We also cleaned a lot of the classes generated by the auth command since those were pretty ugly.

### Removed Helper functions by default

We also removed all instances of helper functions by default since it was confusing developers and was throwing red squiggly marks for text editors. They are still available to be used but they will not be known to developers unless they discover them in the documentation. Now all default code explicitly resolves via the container and helper functions can be used on the developers own terms.

Helper functions are still available but you will need to use them on your own terms.

### Added seeds by default

Now every application has a basic seeding structure setup which is the same as if running the `craft seed` command. This is to promote more use of this awesome feature which can be used in migration files for quick seeding of databases for development.

### Added code to \_\_init\_\_.py file in migrations and seeds

We were previously not able to import code into our migration files or database seeders because the command line tool would not pick up our current working directory to import classes into. Now the migrations module and seeds module have 3 lines of code:

```text
import os
import sys
sys.path.append(os.getcwd())
```

this helpers when running the command line to import code into these modules.

### Route Printing

In development you would see a message like:

```text
GET Route: /dashboard
```

When you hit a route in development mode. Well you would also hit it in production mode too since that was never turned off. Although this is likely fine, it would slow down the framework significantly under load since it takes a bit of resources to print something that didn't need to be printed. This enables a bit of a performance boost.

## Added migrate:status Command

This command gets the statuses of all migrations in all directories. To include third party migration directories that are added to your project.

## Added `simple` container bindings

Sometimes you do not need to bind an object to any key, you just want the object in the container. For this you can now do `simple` bindings like this:

```python
app.simple(Obj())
```

## Added a new global mail helper

This new mail helper can be used globally which points to the default mail driver:

```python
def show(self):
    mail_helper().to(..)
```

## Removed the need for `|safe` filters on built in template helpers.

We no longer need to do:

```markup
{{ csrf_field|safe }}
```

We can now simply do:

```markup
{{ csrf_field }}
```

## Improved setting status codes

Previously we had to specify the status code as a string:

```python
def show(self, request: Request):
    request.status('500 Internal Server Error')
```

in order for these to be used properly. Now we can just specify the status code:

```python
def show(self, request: Request):
    request.status(500)
```

## Added several new methods to service providers

There is quite a bit of things to remember when binding various things into the container. For example when binding commands, the key needs to be postfixed with `Command` like `ModelCommand`. Now we can do things like:

```python
def register(self):
    self.commands(Command1(), Command2())
```

Along with this there are several other methods to help you bind things into the container without having to remember all the special rules involved, if any.

## Added View Routes

We now have View Routes on all instances of the normal HTTP classes:

```python
Get().view('/url', 'some/template', {'key': 'value'})
```

## Renamed cache\_exists to exists

We previously used this method on the Cache class like so:

```python
def show(self, Cache):
    Cache.cache_exists('key')
```

Now we removed the `cache_` prefix and it is just:

```python
from masonite import Cache

def show(self, cache: Cache):
    cache.exists('key')
```

## Added without method to request class

We can now use the `.without()` method on the request class which returns all inputs except the ones specified:

```python
def show(self, request: Request):
    request.without('key1', 'key2')
```

## Added port to database

Previously the port was missing from the database configuration settings. This was fine when using the default connection but did not work unless added to the config.

## Added ability to use a dictionary for setting headers.

Instead of doing something like:

```python
def show(self, request: Request):
    request.header('key1', 'value1')
    request.header('key2', 'value2')
```

We can now use a dictionary:

```python
def show(self, request: Request):
    request.header({
        'key1': 'value1',
        'key2': 'value2'
    })
```

## Added a new Match route

We can now specify a route with multiple HTTP methods. This can be done like so:

```python
from masonite.routes import Match

Match(['GET', 'POST']).route('/url', 'SomeController@show')
```

## Added Masonite Events into core

Core can now emit events that can be listened to through the container.

## Added ability to set email verification

Now you can setup a way to send email verifications into your user signup workflow simply but inherting a class to your User model.

## Request redirection set status codes

Now all redirections set the status code implicitly instead of explicitly needing to set them.

## Added craft middleware command

Now you can use `craft middleware MiddlewareName` in order to scaffold middleware like other classes.

## View can use dot notation

All views can optionally use dot notation instead of foward slashes:

```text
return view.render('some/template/here')
```

is the same as:

```text
return view.render('some.template.here')
```

## Added Swap to container

We can now do container swapping which is swapping out a class when it is resolved. In other words we may want to change what objects are returned when certain objects are resolved. These objects do not have to be in the container in the first place.

## Added a new env function

You can now use a `env` function to automatically type cast your environment variables turning a numeric into an int:

```python
from masonite import env

env('DB_PORT', '5432') #== 5432 (int)
```

## Added ability to resolve with paramaters at the same time

You can now resolve from a container with a parameter list in addition to custom parameters.

## Added password reset to auth command

In addition to all the awesome things that `craft auth` generates, we now generate password reset views and controllers as well for you

## Route Compiler

Fixed an issue where custom route compilers was not working well with request parameters

## Added Database Seeders

All new 2.1 projects have a seeder setup so you can quickly make some mock users to start off your application. All users have a randomly generated email and the password of "secret".

You can run seeders by running:

```text
$ craft seed:run
```

## Made HTTP Prefix to None by Default

When setting headers we had to set the http\_prefix to None more times then not. So it is set by default.

This:

```python
def show(self, request: Request):
    request.header('Content-Type', 'application/xml', http_prefix=None)
```

can change to:

```python
def show(self, request: Request):
    request.header('Content-Type', 'application/xml')
```

## Getting a header returns blank string rather than None

Originally the code:

```python
def show(self, request: Request):
    request.header('Content-Type') #== ''
```

would return `None` if there was no header. Now this returns a blank string.

## Added Maintenance Mode

There is now an up and down command so you can put that in your application in a maintenance state via craft commands:

```text
$ craft down
```

```text
$ craft up
```

There is also a new `MaintenanceModeMiddleware`:

```python
from masonite.middleware import MaintenanceModeMiddleware

HTTP_MIDDLEWARE = [
    ...
    MaintenanceModeMiddleware,
    ...
]
```

## Removed Store Prepend method

We removed the `store_prepend()` method on the upload drivers for the `filename` keyword arg on the store method.

So this:

```
upload.store_prepend('random-string', request.input('file'))
```

now becomes:

```
upload.store(request.input('file'), filename='random-string')
```
