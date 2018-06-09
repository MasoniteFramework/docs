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



