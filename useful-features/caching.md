# Caching

## Introduction

Caching is an important aspect to any project and typically is used to speed up data that never changes and required a lot of resources to get. Powerful caching support is important in any application and Masonite comes with great caching support out of the box.

## Getting Started

We need the `CacheProvider` in order to activate caching with Masonite. We do so simple by going to our `config/application.py` file and adding the Service Provider `masonite.providers.CacheProvider.CacheProvider` to the `PROVIDERS` list.

All configuration settings are inside the `config/cache.py` file. Masonite only comes with a simple `disk` driver which stores all of your cache on the file system.

By default, Masonite will store the cache under the `bootstrap/cache` directory but can be changed simply inside the `DRIVERS` dictionary in the `config/cache.py` file. For example to change from `bootstrap/cache` to the `storage/cache/templates` directory, this will look like:

```python
DRIVERS = {
    'disk': {
        'location': 'storage/cache/templates'
    }
}
```

## Using the Cache

To start using the cache, we can use the `Cache` alias that is loaded into the container from the `CacheProvider` Service Provider. We can retrieve this from the container inside any method that is resolved by the container such as drivers, middleware and controllers. For example we can retrieve it from our controller method like so:

```python
def show(self, Cache):
    Cache # returns the cache class
```

Remember that Masonite uses a Service Container and automatic dependency injection. You can read more about it under the [Service Container](../architectural-concepts/service-container.md) documentation.

### Storing

We can easily store items into the cache by doing:

```python
def show(self, Cache):
    Cache.store('key', 'value')
```

This will create a `bootstrap/cache/key.txt` file which contains a simple `value`.

{% hint style="info" %}
Also note that the directory will be automatically created if it does not exist.
{% endhint %}

### Caching For Time

We may only want to cache something for a few seconds or a few days so we can do something like:

```python
def show(self, Cache):
    Cache.store_for('key', 'value', 5, 'seconds')
```

This will store the cache for 5 seconds. If you try to retrieve this value after 5 seconds, the Cache class will return `None` so be sure to check.

### Getting

It wouldn't be very good if we could only store values and not retrieve them. So we can also do this simple by doing:

```python
def show(self, Cache):
    Cache.get('key')
```

Again this will return `None` if a cache is expired. If there is no time limit on the cache, this will simply always return the cache value.

### Checking Validity

You can also explicitly check if a cache is still valid by doing:

```python
def show(self, Cache):
    Cache.is_valid('key')
```

This will return a boolean if a key is valid. This means it is not expired.

### Checking Cache Exists

We'll have to sometimes check if a cache even exists so we can do that by running:

```python
def show(self, Cache):
    Cache.cache_exists('key')
```

Which will return a boolean if the cache exists or not.

### Updating

We may also want to update a cache. For a real world example, this is used for API's for example when updating the cache for rate limiting. This will not reset the expiration, only update the value.

```python
def show(self, Cache):
    Cache.update('key', 'value')
```

### Deleting

You can delete a cache by key using:

```python
def show(self, Cache):
    Cache.delete('key')
```

