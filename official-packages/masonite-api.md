# Masonite API

## Introduction

Masonite API is a package designed to make it dead simple to add externally facing API's with various types of authentication and permission scopes. There is a new concept called "API Resources" which you will use to build your specific endpoint. In this documentation we will walk through how to make a User Resource so we can walk through the various moving parts.

## Installation

Just run:

```bash
$ pip install masonite-api
```

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
    UserResource('/api/users').routes(),
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
PATCH/PUT /api/user/@id
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

## Overriding Methods

You may want to override some methods that are used internally by the API endpoint to return the necessary data.

The methods are: create, index, show, update, delete.

You can check the repo on how these methods are used and how you should modify them but it's pretty straight forward. The show method is used to show all the results are is what is returned when the endpoint is something like `/api/user`.

Overriding a method will look something like:

```python
from api.resources import Resource
from masonite.request import Request
from app.User import User
from api.serializers import JSONSerializer

class UserResource(Resource, JSONSerializer):
    model = User
    methods = ['create', 'index', 'show']

    def show(self, request: Request):
        return self.model.find(request.id('id'))
```

This will not only return all the results where active is `1`. Keep in mind as well that these methods are resolved via the container so we can use dependency injection:

```python
from api.resources import Resource
from app.User import User
from api.serializers import JSONSerializer
from masonite.request import Request

class UserResource(Resource, JSONSerializer):
    model = User
    methods = ['create', 'index', 'show']

    def show(self, request: Request):
        return self.model.where('active', self.request.input('active')).get()

    def index(self):
        return self.model.all()
```

The index method is ran when getting all records with: `POST /api/user`.

## Removing model attributes

Currently our response may look something like:

```javascript
{
    "id": 1,
    "name": "username",
    "email": "email@email.com",
    "password": "12345",
    "remember_token": null,
    "created_at": "2018-09-23T07:33:30.118068",
    "updated_at": "2018-09-23T11:47:48.962105",
    "customer_id": null,
    "plan_id": null,
    "is_active": null
}
```

You might not want to display all the model attributes like `id`, `email` or `password`. We can choose to remove these with the `without` class attribute:

```python
from api.resources import Resource
from app.User import User
from api.serializers import JSONSerializer
from masonite.request import Request

class UserResource(Resource, JSONSerializer):
    model = User
    methods = ['create', 'index', 'show']
    without = ['id', 'email', 'password']
```

Now our response will look like:

```javascript
{
    "name": "username",
    "remember_token": null,
    "created_at": "2018-09-23T07:33:30.118068",
    "updated_at": "2018-09-23T11:47:48.962105",
    "customer_id": null,
    "plan_id": null,
    "is_active": null
}
```

Yes, it's that easy.

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

Great! If we specify a token by hitting [http://localhost:8000/api/user?token=1234](http://localhost:8000/api/user?token=1234) then we will see a different error:

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

Now we can make a POST request to `http://localhost:8000/token` with your username and password which will give us back a JWT token.

{% api-method method="post" host="http://localhost:8000" path="/token" %}
{% api-method-summary %}
Retrieve a JWT Token
{% endapi-method-summary %}

{% api-method-description %}
Use this endpoint to retrieve new tokens using a username and password. The username and password will default to your regular authentication model located in `config/auth.py`.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-query-parameters %}
{% api-method-parameter name="username" type="string" required=true %}
The username to authenticate using your authentication model
{% endapi-method-parameter %}

{% api-method-parameter name="password" type="string" required=true %}
The password to authenticate using your authentication model
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3N1ZWQiOiIyMDE4LTA5LTIzVDE1OjA1OjM4LjE1MTQyMC0wNDowMCIsImV4cGlyZXMiOiIyMDE4LTA5LTIzVDE1OjEwOjM4LjE1MTY4Mi0wNDowMCIsInNjb3BlcyI6ZmFsc2V9.7BnD7Ro1oK2WdJOU4hsBY3KK0tojBszZAaazQ6MMK-4"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

we can now use this token to make our calls by using that new token. This token is good for 5 minutes and it will be required to refresh the token once expired.

### Refreshing Tokens

Once our JWT token expires, we need to refresh it by sending our old expired token to

{% api-method method="post" host="http://localhost:8000" path="/token/refresh" %}
{% api-method-summary %}
Refresh JWT Tokens
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="token" type="string" required=true %}
The expired JWT token
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3N1ZWQiOiIyMDE4LTA5LTIzVDE1OjA1OjM4LjE1MTQyMC0wNDowMCIsImV4cGlyZXMiOiIyMDE4LTA5LTIzVDE1OjEwOjM4LjE1MTY4Mi0wNDowMCIsInNjb3BlcyI6ZmFsc2V9.7BnD7Ro1oK2WdJOU4hsBY3KK0tojBszZAaazQ6MMK-4"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

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

## Filter Scopes

Filter scopes is an extension of the scopes above. It will filter the data based on the scope level. This is useful if you want a specific scope to have more permission than other scopes.

To do this we can extend our resource with the `FilterScopes` class:

```python
from api.filters import FilterScopes

class UserResource(..., ..., FilterScopes):
    model = User
    methods = ['create', 'index', 'show']
    scopes = ['user:read']
    filter_scopes = {
        'user:read': ['name', 'email'],
        'user:manager': ['id', 'name', 'email', 'active', 'password']
    }
```

Now when you make this request it will return the columns in accordance with the user scopes.

## Creating Authentication Classes

Authentication classes are extremely simple classes. They just need to inherit the BaseAuthentication class and contain 2 methods: `authenticate` and `get_token`.

```python
from api.authentication import BaseAuthentication

class JWTAuthentication(BaseAuthentication):

    def authenticate(self, request: Request):
        """Authenticate using a JWT token
        """

        pass

    def get_token(self):
        """Returns the decrypted string as a dictionary. This method needs to be overwritten on each authentication class.

        Returns:
            dict -- Should always return a dictionary
        """

        pass
```

### Authenticate

This method is resolved via the container. it is important to note that if the authenticate is successful, it should not return anything. Masonite will only look for exceptions thrown in this method and then correlate an error response to it.

For example if we want to return an error because the token was not found, we can raise that exception:

```python
...
from api.exceptions import NoApiTokenFound
...

    def authenticate(self, request: Request):
        """Authenticate using a JWT token
        """

        if not request.input('token'):
            raise NoApiTokenFound
```

Which will result in an error response:

```javascript
{
    "error": "no API token found"
}
```

### Get Token

This method is used to return a dictionary which is the decrypted version of the token. So however your authentication class should decrypt the token, it needs to do so in this method. This all depends on how the token was encrypted in the first place. This may look something like:

```text
from api.exceptions import InvalidToken
...
        def get_token(self):
        """Returns the decrypted string as a dictionary. This method needs to be overwritten on each authentication class.

        Returns:
            dict -- Should always return a dictionary
        """
        try:
            return jwt.decode(self.fetch_token(), KEY, algorithms=['HS256'])
        except jwt.DecodeError:
            raise InvalidToken
```

## Creating Serializers

Serializers are simple classes with a single `serialize` method on them. The `serialize` method takes a single parameter which is the response returned from one of the create, index, show, update, delete methods.

For example if we return something like a model instance:

```python
def index(self):
    return self.model.find(1)
```

We will receive this output into our serialize method:

```python
def serialize(self, response):
    response # == self.model.find(1)
```

which we can then serialize how we need to and return it. Here is an example of a JSON serializer:

```python
import json
from orator.support.collection import Collection
from orator import Model

class JSONSerializer:

    def serialize(self, response):
        """Serialize the model into JSON
        """

        if isinstance(response, Collection):
            return response.serialize()
        elif isinstance(response, Model):
            return response.to_dict()

        return response
```

notice we take the response and then convert that response into a dictionary depending on the response type.

Once we convert to a dictionary here, the `JSONResponseMiddleware` will pick that dictionary up and return a JSON response.

## Builtin Methods

We have access to a few builtin methods from anywhere in our resource.

### Getting the token

The token can be in several different forms coming into the request. It can be either a JWT token or a regular token or some other form of token. Either way it needs to be "unencrypted" in order for it to be used and since the authentication class is responsible for un-encrypting it, it is the responsibility of the authentication class to get the token.

```text
def index(self):
    token = self.get_token()
```

{% hint style="warning" %}
This method is only available when inheriting from an authentication class like `JWTAuthentication` which requires this method and should return the un-encrypted version of the token
{% endhint %}

### Fetching the token

There are a few locations the token can be. The two main locations are in the query string itself \(`/endpoint?token=123..`\) or inside a header \(the `HTTP_AUTHORIZATION` header\). We can get the token regardless of where it is with the `fetch_token()` method:

```text
def index(self):
    token = self.fetch_token()
```

