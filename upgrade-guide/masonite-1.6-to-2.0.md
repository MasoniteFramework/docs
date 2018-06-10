# Masonite 1.6 to 2.0

Masonite 2 brings an incredible new release to the Masonite family. This release brings a lot of new features to Masonite to include new status codes, database seeding, built in cron scheduling, controller constructor resolving, auto-reloading server, a few new internal ways that Masonite handles things, speed improvements to some code elements and so much more. We think developers will be extremely happy with this release.

Upgrading from Masonite 1.6 to Masonite 2.0 shouldn't take very long. On an average sized project, this upgrade should take around 30 minutes. We'll walk you through the changes you have to make to your current project and explain the reasoning behind it.

## Application Configuration 

Masonite 2 adds some improvements with imports. Previously we had to import providers and drivers like:

```python
from masonite.providers.UploadProvider import UploadProvider
```

Because of this, all framework service providers will need to change as well to:

{% code-tabs %}
{% code-tabs-item title="config/application.py" %}
```python
PROVIDERS = [
    # Framework Providers
    'masonite.providers.AppProvider',
    'app.providers.SessionProvider',
    'masonite.providers.RouteProvider',
    # 'entry.providers.ApiProvider',
    'masonite.providers.RedirectionProvider',
    'masonite.providers.StartResponseProvider',
    'masonite.providers.SassProvider',
    'masonite.providers.WhitenoiseProvider',
    ....
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Change all service providers coming from Masonite to these type of strings without the double provider class.

## Duplicate Class Names

With the addition of the above change, any place you have a duplicated class name like:

```python
from masonite.drivers.UploadDriver import UploadDriver
```

You can change it to:

```python
from masonite.drivers import UploadDriver
```

## Redirection

Renamed Request.redirectTo to Request.redirect\_to. Be sure to change any of these instances accordingly.

Removed the .send\(\) method completely on the Request class so all instances of:

```python
def show(self):
    return request().redirect('/dashboard/@id').send({'id': league.id})
```

Need to be changed to:

```python
def show(self):
    return request().redirect('/dashboard/@id', {'id': league.id})
```

## CSRF Middleware

Some variable internals have changed to prepend a double underscore to them to better symbolize they are being handled internally. Because of this we need to change any instances of csrf\_token to \_\_token in  the CSRF Middleware file.

{% hint style="info" %}
You can check for what the class should look like from the [MasoniteFramework/masonite](https://github.com/MasoniteFramework/masonite) repository
{% endhint %}

## Autoloading

Masonite 2 comes with a new autoloader. This can load all classes in any directory you specify right into the service container when the server first starts. This is incredibly useful for loading your models right into the container.

Simply add a new AUTOLOAD constant in your config/application.py file. This is the entire section of the autoload constant.

{% code-tabs %}
{% code-tabs-item title="config/application.py" %}
```python
'''
|--------------------------------------------------------------------------
| Autoload Directories
|--------------------------------------------------------------------------
|
| List of directories that are used to find classes and autoload them into 
| the Service Container. This is initially used to find models and load
| them in but feel free to autoload any directories
|
'''

AUTOLOAD = [
    'app',
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

By default this points to the app directory where models are stored by default but if you moved your models to other directories like app/models or app/admin/models then add those directories to your list:

{% code-tabs %}
{% code-tabs-item title="config/application.py" %}
```python
....

AUTOLOAD = [
    'app',
    'app/models',
    'app/admin/models'
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="danger" %}
Be caution that this will autoload all models into the [Service Container](../architectural-concepts/service-container.md) with the class name as the key and the class as the binding. If you have a class called Request then this will override the Request class which is an unintended side effect.
{% endhint %}

## RedirectionProvider

Because of a minor rewrite of the Request class, we now do not need the RedirectionProvider. You can remove the RedirectionProvider completely in your PROVIDERS list.

## StatusCodeProvider

There is a new status code provider which adds support for adding custom status codes and rendering better default status code pages such as 400 and 500 error pages. This should be added right above the StartResponseProvider:

{% code-tabs %}
{% code-tabs-item title="config/application.py" %}
```python
PROVIDERS = [
    # Framework Providers
    ...
    'masonite.providers.RouteProvider',
    'masonite.providers.RedirectionProvider',
    
    # New provider here above StartResponseProvider
    'masonite.providers.StatusCodeProvider',
    
    'masonite.providers.StartResponseProvider',
    'masonite.providers.WhitenoiseProvider',
    ...
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## .env File

The .env got a small upgrade and in order to make the `APP_DEBUG` variable consistent, it should be set to either `True` or `False`. Previously this was set to something like `true` or `false`.

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
APP_DEBUG=True
# or
APP_DEBUG=False
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Finished

That's it! You're all done upgrading Masonite 1.6 to Masonite 2.0. Build something awesome!

{% hint style="success" %}
Be sure to read about all the [changes in Masonite 2](../whats-new/masonite-2.0.md) to ensure that your application is completely up to date
{% endhint %}



