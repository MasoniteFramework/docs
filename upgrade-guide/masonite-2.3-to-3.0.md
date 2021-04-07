# Masonite 2.3 to 3.0

This guide is designed to give you as much information as possible to upgrade your application from Masonite 2.3 to Masonite 3.0.

We will go through each new breaking change in Masonite and code examples on how to upgrade the code to use Masonite 3. If there is any code breaking during the upgrade, please go to our Slack channel and let us know so we can add more information to this documentation.

This document will be broken down into 2 parts, upgrading from Orator to Masonite and upgrading from Masonite 2.3 to 3.0.

Before you go through this document it is highly recommended that you read [Whats New in Masonite 3](/whats-new/masonite-3.0.md).

Note that there are some large changes from 2.3 to 3.0. Depending on the size of your project it might make more sense to rebuild your project and port your code to a new Masonite 3 app. This guide is for changing your existing projects.

# Upgrading Masonite

## Requirement Changes

We need to uninstall some packages and install some others

First uninstall masonite and orator and install masonite 3:

```
$ pip uninstall masonite
$ pip uninstall orator
$ pip install masonite==3.0
```

That should be all you need to get the requirements up to date. 

At this point Masonite will not work because our codebase is not updated to work with Masonite 3

Now we can start changing the application to get our app to run again.

## Request init Offset

The request class is now not even initialized until a request is sent. Previously the request class acted as a singleton but it is now a new class that is initialized on every request. Because of this change, any place you previously had been fetching the request class before the request is sent will no longer work. This could be in several places but is likely most common to be in any class `__init__` methods that are initialized before a request was sent, like in a service provider where the `wsgi` attribute is `False`. 

## Headers On Response

The response headers are now set on the response class. This makes much more sense now. Previously we set them on the request class which really didn't make sense.

Any place in your code where you want a header to be on the response. The code changes may look something like this:

```python
from masonite.request import Request

class CustomMiddleware(Middleware):
  def __init__(self, request: Request)
      self.request = request
  # ...
  def after(self):
  		self.request.header('IS_INERTIA', 'true')
```

Should now be written as:

```python
from masonite.response import Response

class CustomMiddleware(Middleware):
  def __init__(self, response: Response)
      self.response = response
  # ...
  def after(self):
  		self.response.header('IS_INERTIA', 'true')
```



## HTTP Header Prefix.

Some headers used to have to be prefixed as `HTTP_` to be used correctly. This is no longer required. Code where you have done this:

```python
request.header('HTTP_IS_INERTIA', 'true')
```

Can be changed to:

```python
response.header('IS_INERTIA', 'true')
```

## Status Codes

Status codes used to be set on the request class but is now set on the response class. The method and uses are the same:

```python
request.status(200)
```

Should be changed to:

```python
response.status(200)
```

## New Providers

Because of the request singleton change we had to offset some other instantiation so we added 2 new providers you need to add. The placement of the `RequestHelpersProvider` and `CsrfProvider` needs to be between the `AppProvider` and the `RouteProvider`. The change is in the `config/providers.py` file.

```python
from masonite.providers import RequestHelperProvider

# ...
PROVIDERS = [
  # Framework Providers
  AppProvider, # <-- AppProvider Here
  RequestHelpersProvider, # <-- In the middle
  CsrfProvider, # <-- In the middle
  SessionProvider,
  RouteProvider, # <-- RouteProvider here
```

We also need to add the orm provider from Masonite ORM. It doesn't really matter where add this so we can add it at the bottom of the list somewhere:

```python
from masoniteorm.providers import ORMProvider

PROVIDERS = [
  # ..
  # Third Party Providers
  ORMProvider,
  # ..
]
```

## WSGI.py File Change

There used to be a `bootstrap/start.py` file in Masonite apps. This file contained a method was used to handle the request and response lifecycle. There was no reason to keep the method inside the application since it was such a low level method and was crucial for the framework to work.

So we need to import the method from the Masonite codebase instead of the start.py file. We also renamed the method `response_handler` instead of `app`. Finally environment variables are now loaded at the very beginning.

```diff
+ from masonite.environment import LoadEnvironment

+ LoadEnvironment()

- from bootstrap.start import app
+ from masonite.wsgi import response_handler
from src.masonite.helpers import config

# ..

container = App()

- container.bind('WSGI', app)
+ container.bind('WSGI', response_handler)

container.bind('Container', container)
```

## Craft File

You may not have to make this change but in some previous versions of Masonite, the craft file that came with new projects was wrong. The file is called `craft` and is in the root of your project directory. If the changes in green already exist then your project is correct. If you have the code in red then make this diff change:

```diff
- from masonite import info
+ from masonite import __version__

from wsgi import container

- application = Application('Masonite Version:', info.VERSION)
+ application = Application('Masonite Version:', __version__)
```

## Queue Table

If you use Masonite queues, there are 3 new columns on the `queue_jobs` table. Please make a migration and add these 3 columns:

First make the migration

```
$ python craft migration add_fields_to_queue_jobs_table --table queue_jobs
```

Then add the migration:

```python
  def up(self):
      with self.schema.table("queue_jobs") as table:
          table.string("queue")
          table.timestamp("available_at").nullable()
          table.timestamp("reserved_at").nullable()

  def down(self):
      with self.schema.table("queue_jobs") as table:
          table.drop_column("queue", "available_at", "reserved_at")
```

Then run the migration

```
$ python craft migrate
```

## Dropped route helpers

If you used any helper routes inside your web.py file, these have been removed. You will now need to use only route classes:

If you have code inside your web.py 

```python
from masonite.helpers.routes import get
```

You will need to remove this import and use the class based routes:

```python
from masonite.routes import Get
```

This applied for the helpers: `get`, `post`, `put`, `patch`, `delete` and `group` helpers.

## Flashed Messages

In previous versions of Masonite, Masonite would set flashed messages for a 2 second expiration time. This caused numerous different issues like what happens when a page redirection took longer than 2 seconds or what happens when the client times are not synced correctly.

Now we have took a "get and delete" approach. So now the flash data is deleted when it is retrieved. This means that flash data can stay in session until it is fetched. 

To do this we have a new method for the "get and delete" of flash data.

If you are using the `bag()` helper in your templates then this:

```diff
@if bag().any()
-  @for error in bag().messages()
+  @for error in bag().get_errors()
    <div class="alert alert-danger" role="alert">
        {{ error }}
    </div>
  @endfor
@endif
```

If you are using the `session()` helper than you will need to take a similiar approach:

```diff

@if session().has('errors')
  @for key, error_list in session().get_flashed('errors').items()
    <div class="alert alert-danger" role="alert">
        {{ error }}
    </div>
  @endfor
@endif
```

# Upgrading Orator

For upgrading from Orator to Masonite ORM please read the [Orator to Masonite ORM guide](https://orm.masoniteproject.com/orator-to-masonite-orm)
