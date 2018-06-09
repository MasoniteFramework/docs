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

## Tinker Command

There is a new command that starts a Python shell and imports the container for you already. Test it out to verify that objects are loaded into your container correctly. It's a great debugging tool.

```text
$ craft tinker
```

## Show Routes Command

Masonite 2 ships with an awesome little helper command that allows you to see all the routes in your application

```text
$ craft show:routes
```

## Server Reloading

A huge update to Masonite is the new `--reload` flag on the serve command. Now the server will automatically restart when it detects a file change. You can use the `-r` flag as a shorthand:

```text
$ craft serve -r
```

## Autoloading

An incredible new feature is autoloading support. You can now list directories in the new AUTOLOAD constant in your config/application.py file and it will automatically load all classes into the container. This is great for loading command and models into the container when the server starts up.

## Updated Libraries

Updated all libraries to the latest version with the exception of the Pendulum library which latest version is a breaking change. The breaking change would not be worth it to add the complexity of upgrading so you may upgrade on a per project basis.

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
Read more about changing duplicated class names under the [Duplicate Class Names](../upgrade-guide/masonite-1.6-to-2.0.md#duplicate-class-names) directory 
{% endhint %}

## Redirection Provider

Removed the need for the redirection provider completely. You need to remove this from your PROVIDERS list.

## Redirection

Renamed `Request.redirectTo` to `Request.redirect_to`

