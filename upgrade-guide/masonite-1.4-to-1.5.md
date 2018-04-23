# Masonite 1.4 to 1.5

## Introduction

Masonite 1.5 doesn't bring many file changes to Masonite so this upgrade is fairly straight forward and should take less than 10 minutes.

## Requirements.txt

All requirements are now gone with the exception of the WSGI server \(`waitress`\) and the Masonite dependency. You should remove all dependencies and only put:

```text
waitress==1.1.0
masonite>=1.5,<=1.5.99
```

## Site Packages Configuration

If you have added your site packages directory to our packages configuration file, you can now remove this because Craft commands can now detect your site packages directory in your virtual environment.

## Api Removal

Remove the `masonite.providers.ApiProvider.ApiProvider` from the `PROVIDERS` list as this has been removed completely in 1.5

If you are using the `Api()` route inside `routes/api.py` for API endpoints then remove this as well. You will need to implement API endpoints using the new Official [Masonite Entry](http://entry.masoniteproject.com) package instead.

You'll also have to add a new `RESOURCES = []` line to your `routes/api.py` file for the new Masonite Entry package if you choose to use it.

## Craft Commands

This release works with the new craft command release. Upgrade to version `masonite-cli / 1.1+`. `<1.1` will only work with Masonite 1.4 and below.

Simply run:

```text
$ pip install --upgrade masonite-cli
```

{% hint style="warning" %}
**You may have to run sudo if you are using a UNIX machine.**
{% endhint %}

## Sessions

Masonite 1.5 now has sessions that can be used to hold temporary data. It comes with the cookie and memory drivers. Memory stores all data in a class which is lost when the server restarts and the cookie driver sets cookies in the browser.

There is a new `config/session.py` file you can copy and paste:

```python
''' Session Settings '''

'''
|--------------------------------------------------------------------------
| Session Driver
|--------------------------------------------------------------------------
|
| Sessions are able to be linked to an individual user and carry data from
| request to request. The memory driver will store all the session data
| inside memory which will delete when the server stops running.
|
| Supported: 'memory', 'cookie'
| 
'''

DRIVER = 'memory'
```

As well as add the `SessionProvider` inside your `PROVIDERS` list just below the `AppProvider`:

```python
PROVIDERS = [
    # Framework Providers
    'masonite.providers.AppProvider.AppProvider',

    # New Provider
    'masonite.providers.SessionProvider.SessionProvider',
    'masonite.providers.RouteProvider.RouteProvider',
    ....
]
```

## Finished

That's it! You have officially upgrades to Masonite 1.5

