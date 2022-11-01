Masonite 5 brings new changes ... TODO

# Python Version

Masonite 5 adds support for Python 3.11.

# Install Masonite 5

First step of the upgrade guide is to uninstall Masonite 4 and install Masonite 5:

```
pip uninstall masonite
pip install masonite==5.0.0
```

# Import Path Changes

The import path has changed for the following:

```diff
- from masonite.essentials.helpers import hashid
+ from masonite.helpers import hashid

- from masonite.essentials.middleware import HashIDMiddleware
+ from masonite.hashid.middleware import HashIDMiddleware


```

# Config Changes

## Cache

Rename `memcache` driver to `memcached` to the `config/cache.py` file.

```python
STORES = {
    "memcached": {
        "driver": "memcached",
        "host": "127.0.0.1",
        "port": "11211",
        "password": "",
        "name": "masonite4",
    },
}
```

## Providers

Two new providers have been created, that you will need to add them in `config/providers.py`file.

```python
from masonite.providers import PresetsProvider, SecurityProvider, LoggingProvider

#...

PROVIDERS = [
    FrameworkProvider,
    HelpersProvider,
    SecurityProvider, # insert here
    LoggingProvider, # insert here
    RouteProvider,
    #..
    ValidationProvider,
    PresetsProvider, # insert here
    AuthorizationProvider,
    ORMProvider,
    AppProvider,
]
```



# Cache

Memcached driver name has been fixed. It was called `Memcache` as its real name is `Memcached`.
When using this driver you will now need to access it like this:
```python
self.application.make("cache").store("memcached").get("key")
```
or via the facade:
```python
Cache.store("memcached").get("key")
```

# Helpers

All Masonite helpers can now be imported from `helpers` module to ease development experience.

```python
from masonite.helpers import config, url, env, app, optional, collect, compact
```

A new helper to access application container `app` has been introduced ! More information in [Helpers](../features/helpers.md#app) documentation.
