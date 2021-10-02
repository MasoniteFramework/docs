Masonite provides a powerful caching feature to keep any data cached that may be needless or expensive to fetch on every request.

Masonite caching allows you to save and fetch data, set expiration times and manage different cache stores.

# Adding Cache Data

You can easily add cache data using the`add` method. This will either fetch the data already in the cache, if it is not expired, or it will insert the new value.

```python
def store(self, cache: Cache):
  data = cache.add('age', '21')
```

If `age` key exists in the cache AND it is not expired, then "21" will be added to the cache and returned. If the `age` key does not exist or is not expired then it will return whatever data is in the cache for that key.

# Putting Cache Data

The `add` method will either return data already in the cache or put data into the cache. The `put` method will put data into the cache regardless of if it exists already. This is a good way to overwrite data in the cache:

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

# Getting Cache Data

You can get cache data from the cache. If the data is expired then this will either return `None` or the default you specify:

```python
cache.get('age', '40')
```

This will either fetch the correct age data from the cache or return the default value of `40`.

# Checking Existence

You can also see if a value exists in the cache (and it is not expired):

```python
cache.has('age')
```

# Forgetting a Cache Item

If you want to forget an item in the cache you can:

```python
cache.forget('age')
```

This will remove this item from the cache.

# Increment / Decrement

You can increment and decrement a value if it is an integer:

```python
cache.get('age') #== 21
cache.increment('age') #== 22
cache.decrement('age') #== 21
```

# Remembering

Remembering is a great way to save something to a cache using a callable:

```python
from app.models import User

remember("total_users", lambda cache: (
  cache.put("total_users", User.all().count(), 300)
))
```

# Flushing the Cache

To delete everything in the cache you can simply flush it:

```python
cache.flush()
```



