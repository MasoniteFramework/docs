# Masonite 2.0

Masonite 2 brings an incredible new release to the Masonite family. This release brings a lot of new features to Masonite to include new status codes, database seeding, built in cron scheduling, controller constructor resolving, auto-reloading server, a few new internal ways that Masonite handles things, speed improvements to some code elements and so much more. We think developers will be extremely happy with this release.

Upgrading from Masonite 1.6 to Masonite 2.0 shouldn't take very long. On an average sized project, this upgrade should take around 30 minutes. We'll walk you through the changes you have to make to your current project and explain the reasoning behind it.

{% hint style="success" %}
Checkout the [Upgrade Guide for Masonite 1.6 to 2.0](../upgrade-guide/masonite-1.6-to-2.0.md)
{% endhint %}

## Controller Constructors

Controller constructors are now resolved by the container so this removed some redundancy within your code and any duplicated auto resolving can now be directly in your constructor:

```python
from masonite.request import Request

class YourController:
    def __init__(self, request: Request):
        self.request = Request
    
    def show(self):
        print(self.request) # <class masonite.request.Request>
```

{% hint style="success" %}
Read more in the [Controllers](../the-basics/controllers.md#container-resolving) documentation.
{% endhint %}

## Tinker Command

There is a new command that starts a Python shell and imports the container for you already. Test it out to verify that objects are loaded into your container correctly. It's a great debugging tool.

```text
$ craft tinker
```

{% hint style="success" %}
Read more in [The Craft Command Introduction](../the-craft-command/introduction.md#tinker-command) documentation.
{% endhint %}

## Show Routes Command

Masonite 2 ships with an awesome little helper command that allows you to see all the routes in your application

```text
$ craft show:routes
```

{% hint style="success" %}
Read more in [The Craft Command Introduction](../the-craft-command/introduction.md#show-routes-command) documentation.
{% endhint %}

## Server Reloading

A huge update to Masonite is the new `--reload` flag on the serve command. Now the server will automatically restart when it detects a file change. You can use the `-r` flag as a shorthand:

```text
$ craft serve -r
```

{% hint style="success" %}
Read more in [The Craft Command](../the-craft-command/introduction.md#running-the-wsgi-server) Introduction documentation.
{% endhint %}

## Autoloading

An incredible new feature is autoloading support. You can now list directories in the new `AUTOLOAD` constant in your `config/application.py` file and it will automatically load all classes into the container. This is great for loading command and models into the container when the server starts up.

You can also use this class as a standalone class in your own service providers.

{% hint style="success" %}
Read more in [Autoloading](../advanced/autoloading.md) documentation.
{% endhint %}

## Updated Libraries

Updated all libraries to the latest version with the exception of the Pendulum library which latest version is a breaking change and therefore was left out. The breaking change would not be worth it to add the complexity of upgrading so you may upgrade on a per project basis.

## Removed Importing Duplicate Class Names

Previously you had to import classes like:

```text
from masonite.drivers.UploadDriver import UploadDriver
```

Now you can simply specify:

```text
from masonite.drivers import UploadDriver
```

Because of this change we no longer need the same duplicated class names in the PROVIDERS list either.

{% hint style="success" %}
Read more about changing duplicated class names under the [Duplicate Class Names](../upgrade-guide/masonite-1.6-to-2.0.md#duplicate-class-names) documentation.
{% endhint %}

## Redirection Provider

Removed the need for the redirection provider completely. You need to remove this from your `PROVIDERS` list.

## Redirection

Renamed `Request.redirectTo` to `Request.redirect_to`

Also removed the .send\(\) method and moved the dictionary into a parameter:

```python
def show(self):
    return request().redirect('/dashboard/@id', {'id': '5'})
```

{% hint style="success" %}
Read more in the [Requests](../the-basics/requests.md#redirection) documentation.
{% endhint %}

## Request Only

Added a new Request.only method to fetch only specific inputs needed.

{% hint style="success" %}
Read more in [Requests](../the-basics/requests.md#only) documentation.
{% endhint %}

## Get Request Method

Added a new `Request.get_request_method()` method to the `Request` class.

{% hint style="success" %}
Read more in [Requests](../the-basics/requests.md#get-request-method-type) documentation.
{% endhint %}

## New Argument in Request.all

You can now completely remove fetching of any inputs that Masonite handles internally such as \_\_token and \_\_method when fetching any inputs. This is also great for building third party libraries:

```python
Request.all(internal_variables=False)
```

{% hint style="success" %}
Read more in [Requests](../the-basics/requests.md#input-data) documentation.
{% endhint %}

## Made several changes to the CSRF Middleware

Because of the changes to internal framework variables, there are several changes to the CSRF middleware that comes in every application of Masonite. 

{% hint style="success" %}
Be sure to read the changes in the [Upgrade Guide 1.6 to 2.0](../upgrade-guide/masonite-1.6-to-2.0.md).
{% endhint %}

## Added Scheduler to Masonite

Added a new default package to Masonite that allows scheduling recurring tasks:

{% hint style="success" %}
Read about Masonite Scheduler under the [Task Scheduling](../useful-features/task-scheduling.md) documentation.
{% endhint %}

## Added Database Seeding Support

It's important during development that you have the ability to seed your database with dummy data. This will improve team development with Masonite to get everyones database setup accordingly.

{% hint style="success" %}
Read more in the [Database Seeding](../advanced/database-seeding-incomplete.md) documentation.
{% endhint %}

## Added a New Static File Helper

Now all templates have a new static function in them to improve rendering of static assets

{% hint style="success" %}
Read more in the [Static Files](../the-basics/static-files.md) documentation.
{% endhint %}

## Added a New Password Helper

You can use the password helper to hash passwords more simply than using straight bcrypt:

```python
from masonite.helpers import password

password('secret') # returns bcrypt password
```

{% hint style="success" %}
Read more in the [Encryption](../security/encryption.md#hashing-passwords) documentation.
{% endhint %}

## Added Dot Notation To Upload Drivers And Dictionary Support To Driver Locations.

You can now specify which location in your drivers you want to upload to using a new dot notation:

```text
Upload.store(request().input('file'), 'disk.uploads')
```

This will use the directory stored in:

```python
DRIVERS = {
  'disk': {
    'uploads': 'storage/uploads',
    'profiles': 'storage/static/users/profiles/images'
  },
  ...
}
```

{% hint style="success" %}
Read more in the [Uploading](../useful-features/uploading.md#dot-notation) documentation.
{% endhint %}

## Added Status Code Provider

Masonite 2 removes the bland error codes such as 404 and 500 errors and replaces them with a cleaner view. This also allows you to add custom error pages.

{% hint style="success" %}
Read more in the [Status Codes](../advanced/status-codes-incomplete.md) documentation.
{% endhint %}

## Added Explicitly Imported Providers

Providers are now explicitly imported at the top of the file and added to your PROVIDERS list which is now located in `config/providers.py`. This completely removes the need for string providers and boosts the performance of the application sustantially

