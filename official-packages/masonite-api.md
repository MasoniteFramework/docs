# Masonite API

## Introduction

Masonite API is a package designed to make it dead simple to add externally facing API's with various types of authentication and permission scopes. There is a new concept called "API Resources" which you will use to build your specific endpoint. In this documentation we will walk through how to make a User Resource so we can walk through the various moving parts.

## Creating a Resource

We can create API Resources by building them wherever you want to. In this documentation we will put them in `app/resources`. We just need to create a simple resource class which inherits from `api.resources.Resource`.

```python
from api.resources import Resource

class UserResource(Resource):
    pass
```

You should also specify a model by importing it and specifying the model attribute:

```python
from api.resources import Resource
from app.User import User

class UserResource(Resource):
    model = User
```

Lastly for simplicity sake, we can specify a serializer. This will take any Orator models or Collection we return and serialize them into a dictionary which will be picked up via the `JsonResponseMiddleware`. 

```python
from api.resources import Resource
from app.User import User
from api.serializers import JSONSerializer

class UserResource(Resource, JSONSerializer):
    model = User
```

Awesome! Now we are ready to go!

## Specifying our routes

Our resource has several routes that it will build based on the information we provided so let's import it into our web.py file so it will build the routes for us. We can also specify the base route which will be used for all routes. 

```text
...
from app.resources.UserResource import UserResource

ROUTES = [
    ...
    UserResource('/api/user').routes(),
    ...
]
```

This will build a route list that looks like:

```text
========  =============  =======  ========  ============
Method    Path           Name     Domain    Middleware
========  =============  =======  ========  ============
POST      /api/user
GET       /api/user
GET       /api/user/@id
PUT       /api/user/@id
DELETE    /api/user/@id
========  =============  =======  ========  ============
```

We can also specify the routes that we want to create by setting the `methods` attribute:

```python
from api.resources import Resource
from app.User import User
from api.serializers import JSONSerializer

class UserResource(Resource, JSONSerializer):
    model = User
    methods = ['create', 'index', 'show']
```

This will only build those routes dependent on the methods:

```text
========  =============  =======  ========  ============
Method    Path           Name     Domain    Middleware
========  =============  =======  ========  ============
POST      /api/user
GET       /api/user
GET       /api/user/@id
========  =============  =======  ========  ============
```

