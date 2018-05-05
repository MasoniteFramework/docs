# Masonite 1.3 to 1.4

## Introduction

Masonite 1.4 brings several new features and a few new files. This is a very simple upgrade and most of the changes were done in the pip package of Masonite. **The upgrade from 1.3 to 1.4 should take less than 10 minutes**

## Requirements.txt File

This requirement file has the `masonite>=1.3,<=1.3.99` requirement. This should be changed to `masonite>=1.4,<=1.4.99`. You should also run `pip install --upgrade -r requirements.txt` to upgrade the Masonite pip package.

## New Cache Folder

There is now a new cache folder under `bootstrap/cache` which will be used to store any cached files you use with the caching feature. Simply create a new `bootstrap/cache` folder and optionally put a `.gitignore` file in it so your source control will pick it up.

## New Cache and Broadcast Configuration

Masonite 1.4 brings a new `config/cache.py` and `config/broadcast.py` files. These files can be found on the [GitHub](https://github.com/MasoniteFramework/masonite) page and can be copied and pasted into your project. Take a look at the new [config/cache.py](https://github.com/MasoniteFramework/masonite/blob/v1.4/config/cache.py) file and the [config/broadcast.py](https://github.com/MasoniteFramework/masonite/blob/v1.4/config/broadcast.py) file. Just copy and paste those configuration files into your project.

## 3 New Service Providers

Masonite comes with a lot of out of the box functionality and nearly all of it is optional but Masonite 1.4 ships with three new providers. Most Service Providers are not ran on every request and therefore does not add significant overhead to each request. To add these 3 new Service Providers simple add these to the bottom of the list of framework providers:

```python
PROVIDERS = [
    # Framework Providers
    ...
    'masonite.providers.HelpersProvider.HelpersProvider',
    'masonite.providers.QueueProvider.QueueProvider',

    # 3 New Providers in Masonite 1.4
    'masonite.providers.BroadcastProvider.BroadcastProvider',
    'masonite.providers.CacheProvider.CacheProvider',
    'masonite.providers.CsrfProvider.CsrfProvider',

    # Third Party Providers

    # Application Providers
    'app.providers.UserModelProvider.UserModelProvider',
    'app.providers.MiddlewareProvider.MiddlewareProvider',
]
```

Note however that if you add the `CsrfProvider` then you will also need the CSRF middleware which is new in Masonite 1.4. Read the section below to add the middleware

## CSRF and CSRF Middleware

Masonite 1.4 adds CSRF protection. So anywhere there is any POST form request, you will need to add the `{{ csrf_field|safe }}` to it. For example:

```markup
<form action="/dashboard" method="POST">
    {{ csrf_field|safe }}
    <input type="text" name="first_name">
</form>
```

This type of protection prevents cross site forgery. In order to activate this feature, we also need to add the [CSRF middleware](https://github.com/MasoniteFramework/masonite/blob/master/app/http/middleware/CsrfMiddleware.py). Copy and paste the middleware into your project under the `app/http/middleware/CsrfMiddleware.py` file.

Lastly, put that middleware into the `HTTP_MIDDLEWARE` list inside `config/middleware.py` like so:

```python
HTTP_MIDDLEWARE = [
    'app.http.middleware.LoadUserMiddleware.LoadUserMiddleware',
    'app.http.middleware.CsrfMiddleware.CsrfMiddleware',
]
```

## Changes to Database Configuration

There has been a slight change in the constants used in the [config/database.py](https://github.com/MasoniteFramework/masonite/blob/master/config/database.py) file. Mainly just for consistency and coding standards. Your file may have some slight changes but this change is optional. If you do make this change, be sure to change any places in your code where you have used the Orator Query Builder. For example any place you may have:

```python
from config import database

database.db.table(...)
```

should now be:

```python
from config import database

database.DB.table(...)
```

with this change

