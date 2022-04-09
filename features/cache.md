Masonite provides a powerful caching feature to keep any data cached that may be needless or expensive to fetch on every request. Masonite caching allows you to save and fetch data, set expiration times and manage different cache stores.

Masonite supports the following cache drivers: [Redis](#redis), [Memcached](#memcached) and a basic [File](#file-cache) driver.

We'll walk through how to configure and use cache in this documentation.

# Configuration

Cache configuration is located at `config/cache.py` file. In this file, you can specify different
cache stores with a name via the `STORES` dictionary and the default to use in your application with the `default` key.

Masonite is configured to use the File cache driver by default, named `local`.

{% code title="config/cache.py" %}
```python
STORES = {
    "default": "local",
    "local": {
        "driver": "file",
        "location": "storage/framework/cache"
    },
    "redis": {
        "driver": "redis",
        "host": "127.0.0.1",
        "port": "6379",
        "password": "",
        "name": "masonite4",
    },
    # ...
}
```
{% endcode %}

{% hint style="info" %}
For production applications, it is recommended to use a more efficient driver such as `Memcached` or `Redis`.
{% endhint %}

## File Cache

File cache driver is storing data by saving it on the server's filesystem. It can be used as is
without third party service.

Location where Masonite will store cache data files defaults to `storage/framework/cache` in projet
root directory but can be changed with `location` parameter.

## Redis

Redis cache driver is requiring the `redis` python package, that you can install with:
```bash
pip install redis
```

Then you should define Redis as default store and configure it with your Redis server parameters:
```python
STORES = {
    "default": "redis",
    "redis": {
        "driver": "redis",
        "host": env("REDIS_HOST", "127.0.0.1"),
        "port": env("REDIS_PORT", "6379"),
        "password": env("REDIS_PASSWORD"),
        "name": env("REDIS_PREFIX", "project name"),
    },
}
```

Finally ensure that the Redis server is running and you're ready to [start using cache](#using-the-cache).


## Memcached

Memcached cache driver is requiring the `pymemcache` python package, that you can install with:

```bash
pip install pymemcache
```

Then you should define Memcached as default store and configure it with your Memcached server parameters:

```python
STORES = {
    "default": "memcache",
    "memcache": {
        "driver": "memcache",
        "host": env("MEMCACHED_HOST", "127.0.0.1"),
        "port": env("MEMCACHED_PORT", "11211"),
        "password": env("MEMCACHED_PASSWORD"),
        "name": env("MEMCACHED_PREFIX", "project name"),
    },
}
```

Finally ensure that the Memcached server is running and you're ready to [start using cache](#using-the-cache).


# Using the Cache

You can access Cache service via the `Cache` facade or by resolving it from the [Service Container](architectural-concepts/service-container.md).

## Storing Data

Two methods are available: `add` and `put`.

### add
You can easily add cache data using the`add` method. This will either fetch the data already in the cache, if it is not expired, or it will insert the new value.

```python
from masonite.cache import Cache

def store(self, cache: Cache):
  data = cache.add('age', '21')
```

If `age` key exists in the cache AND it is not expired, then "21" will be added to the cache and returned. If the `age` key does not exist or is not expired then it will return whatever data is in the cache for that key.

### put

The `put` method will put data into the cache regardless of if it exists already. This is a good way to overwrite data in the cache:

```python
cache.put('age', '21')
```

You can specify the number of seconds that the cache should be valid for. Do not specify any time or specify `None` to keep the data forever.

```python
cache.put('age', '21', seconds=300) # stored for 5 minutes
```

You may also cache lists and dictionaries which will preserve the data types:

```python
cache.put('user', {"name": "Joe"}, seconds=300) # stored for 5 minutes
cache.get("user")['name'] #== "Joe"
```

## Getting Data

You can get cache data from the cache. If the data is expired then this will either return `None` or the default you specify:

```python
cache.get('age', '40')
```

This will either fetch the correct age data from the cache or return the default value of `40`.

## Checking Data Exists

You can also see if a value exists in the cache (and it is not expired):

```python
cache.has('age')
```

## Forgetting Data

If you want to forget an item in the cache you can:

```python
cache.forget('age')
```

This will remove this item from the cache.

## Increment / Decrement Value

You can increment and decrement a value if it is an integer:

```python
cache.get('age') #== 21
cache.increment('age') #== 22
cache.decrement('age') #== 21
```

## Remembering

Remembering is a great way to save something to a cache using a callable:

```python
from app.models import User

remember("total_users", lambda cache: (
  cache.put("total_users", User.all().count(), 300)
))
```

## Flushing the Cache

To delete everything in the cache you can simply flush it:

```python
cache.flush()
```
