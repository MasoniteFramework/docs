Masonite 4 is the biggest change in a Masonite release we have ever had. Smaller applications may benefit from creating a new app and then copying and pasting controllers, routes and views to the new installation.

For medium to large scale applications, you will need to go through the codebase and upgrade everything to use the new structures we have available in Masonite 4.

It is highly recommended that you start at Masonite 3 before upgrading to Masonite 4. If you are running any version less than Masonite 3 then please use the upgrade guide to upgrade to each version first.

# Python Version

Masonite 4 drops support for Python 3.6. You will have to upgrade to Python 3.7+ in order to run Masonite

# Install Masonite 4

First step of the upgrade guide is to uninstall Masonite 3 and install Masonite 4:

```
$ pip uninstall masonite
$ pip install masonite==4.0.0
```

# Craft File

The next step of the upgrade guide is to replace your craft file. This is a basic file in the root of your project that will be used whenever you run `python craft`. We will use this to keep testing our server:

Go to [this file](https://github.com/MasoniteFramework/cookie-cutter/blob/4.0/craft) and copy and paste it into your own craft file in the root of your project. If you are running Masonite 3 then you should already have this file.

# Kernel File

Masonite 4 has a new `Kernel.py` file which is used to customize your application and contains necessary bootstrapping to start your application.

You will also need to put this into the base of your project.

Go to [this file](https://github.com/MasoniteFramework/cookie-cutter/blob/4.0/Kernel.py) and paste it into your own `Kernel.py` file in the root of your project.

Now go through this file and customize any of the locations. Masonite 4 uses a different file structure than Masonite 3. For example, Masonite 3 put all views in `resources/templates` while Masonite 4 has them just in `templates`.

Because of this, you can either change where the files are located by moving the views to a new `templates` directory or you can change the path they are registered in your Kernel:

Go to your `register_configurations` method in your Kernel and inside your `register_templates` method you can change it to

```python
def register_templates(self):
  self.application.bind("views.location", "resources/templates")
```

Go through the rest of the methods and make sure the paths are set correctly.

# Providers

Add the following providers to your project:

```python
from masonite.providers import FrameworkProvider, HelpersProvider, ExceptionProvider, EventProvider, HashServiceProvider

PROVIDERS = [
	FrameworkProvider,
  ExceptionProvider,
  EventProvider,
  HashServiceProvider,
  
```



# Routes

Routes have changed slightly.

First, routes are now all under the `Route` class. The route classes have moved into methods:

```diff
from masonite.routes import Route
ROUTES = [
-     Get('/', 'Controller@method').name('home')
-     Post('/login', 'Controller@method').name('home')
+     Route.get('/', 'Controller@method').name('home')
+     Route.post('/', 'Controller@method').name('home')
]
```

# WSGI File

The WSGI file has also changed.

Go to [this file](https://github.com/MasoniteFramework/cookie-cutter/blob/4.0/wsgi.py) and replace your own `wsgi.py` file

# Import Path Changes

The import path has changed for the following:

```diff
- from masonite.helpers import random_string
+ from masonite.utils.str import random_string

- from masonite import Mail
+ from masonite.mail import Mail

- from masonite.view import View
+ from masonite.views import View

- from masonite.auth import Auth
+ from masonite.authentication import Auth

- from masonite import env
+ from masonite.environment import env

- from masonite.helpers import password
+ from masonite.facades import Hash #== See Hashing documentation

- from masonite.middleware import CsrfMiddleware as Middleware
+ from masonite.middleware import VerifyCsrfToken as Middleware

- from masonite.helpers import config
+ from masonite.configuration import config

- from masonite.drivers import Mailable
+ from masonite.mail import Mailable

- from masonite.provider import ServiceProvider
+ from masonite.providers import Provider
```

# Request and Response redirects

Previously when we had to do a redirection we would use the request class:

```python
def store(self, request: Request):
	return request.redirect('/home')
```

This has now been changed to the response class:

```python
from masonite.response import Response

def store(self, response: Response):
	return response.redirect('/home')
```

Same applies to back redirection:

```diff
- return request.back()
+ return response.back()

```

# Middleware

Middleware has been moved to this new Kernel file. Middleware now works a little different in M4. Middleware has changed in the following ways:

1. middleware no longer needs an `__init__` method. 
2. Middleware requires the request and response parameters inside the before and after methods.
3. Middleware requires either the request or response to be returned

Middleware will change in the following example:

Old:

```python
class AdminMiddleware:
		def __init__(self, request: Request):
        """Inject Any Dependencies From The Service Container.
        Arguments:
            Request {masonite.request.Request} -- The Masonite request object
        """
        self.request = request

    def before(self):
      if not optional(self.request.user()).admin == 1:
            self.request.redirect('/')
    
    def after(self):
      pass
```

To this:

```python
class AdminMiddleware:

    def before(self, request, response):
      if not optional(request.user()).admin == 1:
            return response.redirect('/')
    
    def after(self, request, response):
      pass
```

# User Model & Authentication

Masonite 4 uses the same Masonite ORM package as Masonite 3 but changes a lot of the authentication.

Inherit a new `Authenticates` class

```python
from masonite.authentication import Authenticates
from masoniteorm.models import Model

class User(Model, Authenticates):
  # ..
```

Authentication has also changes slightly. Whenever you are logging in a user the following UI has changed:

```diff
- auth.login(request.input('email'), request.input('password'))
+ auth.attempt(request.input('email'), request.input('password'))
```

# Controllers

Controllers have not changes much but in order for routes to pick up your string controllers, you **must** inherit from Masonite controller class:

```python
from masonite.controllers import Controller

class DashboardController(Controller):
  # ..
```

The rest of your controller structure remains the same.

# Routes

# Config Changes

## Application

Add the following to the `config/application.py` file.

```python
HASHING = {
    "default": "bcrypt",
    "bcrypt": {"rounds": 10},
    "argon2": {"memory": 1024, "threads": 2, "time": 2},
}
```

## Auth

Change the `AUTH` constant to the new `GUARDS` configuration:

```python
GUARDS = {
    "default": "web",
    "web": {"model": User},
    "password_reset_table": "password_resets",
    "password_reset_expiration": 1440,  # in minutes. 24 hours. None if disabled
}
```

Add a new config/exceptions.py file:

```python
HANDLERS = {"stack_overflow": True, "solutions": True}
```

## Storage -> Filesystem

The `config/storage.py` file has been replaced with a `config/filesystem.py` file:

Go to [this file](https://github.com/MasoniteFramework/cookie-cutter/blob/4.0/config/filesystem.py) and copy it into your project. Then move the `STATICFILES` from your storage config into this new filesystem config

## Session

Change the `config/session.py` to the following:

```python
DRIVERS = {
    "default": "cookie",
    "cookie": {},
}
```

## Middleware

After you upgrade all your middleware, you will need to move them from the config/middleware.py file to the top of your Kernel file:

```python
from some.place import CustomMiddleware
class Kernel:

    # ...
    route_middleware = {"web": [
        # ...
        CustomMiddleware
    ],
```

# Session

`get_flashed` method has changed to just `get`. Here is an example in your templates:

```diff
- {{ session().get_flashed('success') }}
+ session().get('success')
```

# Static Helper

The static helper has changed to `asset`:

```diff
- {{ static('s3.uploads', 'invoice.pdf') }}
+ {{ asset('s3.uploads', 'invoice.pdf') }}
```

Finally after all the above changes attempt to run your server:

```terminal
$ python craft serve
```

