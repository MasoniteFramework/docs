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

This will only build those routes dependent on the methods specified:

```python
========  =============  =======  ========  ============
Method    Path           Name     Domain    Middleware
========  =============  =======  ========  ============
POST      /api/user
GET       /api/user
GET       /api/user/@id
========  =============  =======  ========  ============
```

## Authentication

For any API package to be awesome, it really needs to have powerful and simple authentication. 

### JWT Authentication

We can specify our authentication for our specific endpoint by inheriting from a class:

```python
from api.resources import Resource
from app.User import User
from api.serializers import JSONSerializer
from api.authentication import JWTAuthentication

class UserResource(Resource, JSONSerializer, JWTAuthentication):
    model = User
    methods = ['create', 'index', 'show']
```

Now our endpoint is being JWT Authentication. If we hit our endpoint now in the browser by sending a GET request to `http://localhost:8000/api/user`. we will see:

```javascript
{
    "error": "no API token found"
}
```

Great! If we specify a token by hitting http://localhost:8000/api/user?token=1234 then we will see a different error:

```javascript
{
    "error": "token is invalid"
}
```

### JWT Tokens

We can easily create an endpoint for giving out and refreshing API tokens by adding some routes to our web.py:

```python
...
from app.resources.UserResource import UserResource
from api.routes import JWTRoutes

ROUTES = [
    ...
    UserResource('/api/user').routes(),
    JWTRoutes('/token'),
    ...
]
```

Now we can make a POST request to `http://localhost:8000/token` which will give us back a JWT token:

```javascript
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3N1ZWQiOiIyMDE4LTA5LTIzVDE1OjA1OjM4LjE1MTQyMC0wNDowMCIsImV4cGlyZXMiOiIyMDE4LTA5LTIzVDE1OjEwOjM4LjE1MTY4Mi0wNDowMCIsInNjb3BlcyI6ZmFsc2V9.7BnD7Ro1oK2WdJOU4hsBY3KK0tojBszZAaazQ6MMK-4"
}
```

we can now use this token to make our calls by using that new token. This token is good for 5 minutes and it will be required to refresh the token once expired.

## Permission Scopes

You can also specify any permission scopes you need. Most of the time some API endpoints will need to have more restrictive scopes than others. We can specify whatever scopes we need to with the `scopes` attribute as well as inheriting another class.

```python
from api.resources import Resource
from app.User import User
from api.serializers import JSONSerializer
from api.authentication import JWTAuthentication, PermissionScopes

class UserResource(Resource, JSONSerializer, JWTAuthentication, PermissionScopes):
    model = User
    methods = ['create', 'index', 'show']
    scopes = ['user:read']
```

Now all API endpoint will need to have the correct permission scopes. Hitting this API endpoint now will result in:

```javascript
{
    "error": "Permission scope denied. Requires scopes: user:read"
}
```

We can request the scopes we need by spending a POST request back to the endpoint with a scopes input. The scopes should be comma separated. The request should look like:

```text
http://localhost:8000/token?scopes=user:read,user:create
```

This will generate a new token with the correct permission scopes.



