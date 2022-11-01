# Hash ID's

The Masonite hashing feature is a very useful feature to prevent exposing ID's in your application.

Many times you need to expose your database primary keys to the frontend. For example, when updating a record, you might need to pass in a primary key value to a URL like `/users/10/settings`.

Typically you want to hide these key values from a hacker trying to change these values easily.

With the Masonite hashing feature you can change a value like `10` to `l9avmeG` and prevent exposing those sensitive integer values.

## Setup

The Masonite hashing feature automatically decodes values before they get to the controller. To do this it you need to specify both a middleware to help decode the values as well as the provider to register the helper in the templates.

For the middleare you can add it easily:

```python
from masonite.hashid.middleware import HashIDMiddleware

# ..
route_middleware = {
  "web": [
    HashIDMiddleware,
    SessionMiddleware,
    EncryptCookies,
    VerifyCsrfToken,
  ],
}
```

You should put the Hash ID middleware towards the top of the middleware stack so it will decode the request properly before getting to the other middleware in the stack.

The provider should also be added:

```python
from masonite.providers import HashIDProvider

PROVIDERS = [
  # ..
  HashIDProvider,
]
```

This will register a template helper and some other useful features.

## Helper

You can use the helper directly to encode or decode integers easily:

```python
from masonite.helpers import hashid

def show(self):
  hashid(10) #== l9avmeG
  hashid('l9avmeG', decode=True) #== 10
```

## Templates

Inside your templates you can use the `hashid` template helper:

```markup
<!-- user.id == 10 -->
<input type="hidden" name="user_id" value="{{ hashid(user.id) }}"
```

When submitted to the backend the will now be the normal integer value of the user id:

```python
def store(self, request: Request):
  request.input("user_id") #== 10
```

## Route Parameters

When using the template helper, you may also use the hashid feature for request params:

If you have a route like this:

```python
Route.get('/user/@user_id/updates', 'Controller@method')
```

```markup
<!-- user.id == 10 -->
<form action="/user/{{ user.id }}/update" method="POST">
</form>
```

Inside your controller you are able to get the unhashed request parameter:

```python
def store(self, request: Request):
  request.param('user_id') #== 10
```
