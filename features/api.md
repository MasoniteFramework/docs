Adding API Support to your Masonite project is very simple. Masonite comes with supporting for returning JSON responses already but there are a few API features for authenticating and authorizing that are helpful.

Default projects don't come with these features so you'll have to simply register them.

# Getting Started

## Provider

First, register the ApiProvider in your project by adding the service provider to your PROVIDERS list:

```python
from masonite.api.providers import ApiProvider

PROVIDERS = [
    #..
    ApiProvider
]
```

This will register some required classes used for API development.

## Model and Migrations

Next, you must choose a model you want to be responsible for authentication. This is typically your User model. You will have to inherit the `AuthenticatesTokens` class onto this model:

```python
from masoniteorm.models import Model
from masonite.api.authentication import AuthenticatesTokens

class User(Model, AuthenticatesTokens):
    #..
```

This will allow the model to save tokens onto the table.

Next you will add a column to the models table for saving the token. Here we are naming it `api_token` but this is configurable by adding a `__TOKEN_COLUMN__` attribute to your model. Your migration file should look like this:

```python
with self.schema.table("users") as table:
    table.string("api_token").nullable()
```

Then migrate your migrations by running:

```
$ python craft migrate
```

## Configuration

Next, you can create a new API config file. You can do so simply by running the install command:

```python
python craft api:install
```

This will create a new `api.py` config file in your configuration directory that looks like this:

```python
"""API Config"""
from app.models.User import User
from masonite.environment import env

DRIVERS = {
    "jwt": {
        "algorithm": "HS512",
        "secret": env("JWT_SECRET"),
        "model": User,
        "expires": None,
        "authenticates": False,
        "version": None,
    }
}
```

This will attempt to import your user model but if you have a different model you can change it in this file.

This command will also generate a secret key, you should store that secret key in an environment variable called `JWT_SECRET`. This will be used as a salt for encoding and decoding the JWT token.

The `authenticates` key is used as a check to check against the database on every request to see if the token is set on the user. By default, the database is not called to check if the token is valid. One of the benefits of JWT is the need to not have to make a database call to validate the user but if you want that behavior, you can set this option to `True`



## Routes

You should add some routes to your `web.py` file which can be used to authenticate users to give them JWT tokens:

```python
from masonite.api import Api

ROUTES += [
    #.. web routes
]

ROUTES += Api.routes(auth_route="/api/auth", reauth_route="/api/reauth")
```

The above parameters are the defaults already so if you don't want to change them then you don't have to specify them.

You should now create an `api.py` file to add your routes to:

```python
# routes/api.py

ROUTES = [
    Route.get('/users', 'UsersController@index')
]
```

Any routes inside this file will be grouped inside an api middleware stack.

You will then have to add a binding to the container for the location of this file. You can do so in your `Kernel.py` file inside the `register_routes` method:

```python
def register_routes(self):
    #..
    self.application.bind("routes.api.location", "routes/api")
```

## Middleware

Since the routes in your `api.py` file are wrapped in an `api` middleware, you should add a middleware stack in your route middleware in your Kernel file:

```python
# Kernel.py

from masonite.api.middleware import JWTAuthenticationMiddleware
#.. 

class Kernel:

    # ..

    route_middleware = {
        # ..
        "api": [
            JWTAuthenticationMiddleware
        ],
    }
```

This middleware will allow any routes set on this stack to be protected by JWT authorization. 

> By default, all routes in the `routes/api.py` file already have the `api` middleware stack on them so there is no need to specify the stack on all your API routes.

Once these steps are done, you may now create your API's

# Creating Your API

Once the setup is done, we may start building the API.

## Route Resources

One of the ways to build an API is using controller and route resources.

A controller resource is a controller with several methods on them used to specify each action within an entity in the application (like users).

To create a controller resource you can run the controller command with an `-a` flag:

```
$ python craft controller api/UsersController -a
```

This will create a controller with the following methods:

* index
* show
* store
* update
* destroy

You can then create a route resource:

```python
# routes/api.py
from masonite.routes import Route

ROUTES = [
    # ..
    Route.api('users', "api.UserController")
]
```

This will create the below routes which will match up with your API Controller methods:

|  Method	| URL 	|  Action	| Route Name 	|
|---|---|---|---|
| GET 	| /users 	| index 	| users.show 	| 
| GET 	| /users/@id 	| show 	| users.show 	| 
| POST 	| /users 	| store 	| users.store 	| 
| PUT/PATCH 	| /users/@id 	| update 	| users.update 	| 
| DELETE 	| /users/@id 	| destroy 	| users.destroy 	| 

## Authentication

The routes we added earlier contain 2 authentication methods. The `/api/auth` route can be used to get a new authentication token:

First, send a `POST` request with a `username` and `password` to get back a JWT token:

```json
{
    "data": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJleHBpcmVzIjpudWxsLCJ2ZXJzaW9uIjpudWxsfQ.OFBijJFsVm4IombW6Md1RUsN5v_btPwl-qtY-QSTBQ0b7N2pca8BnnT4OjfXOVRrKCWaKUM3QsGj8zqxCD4xJg"
}
```

You should then send this token with either a `token` input or a `Authorization` header:

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJleHBpcmVzIjpudWxsLCJ2ZXJzaW9uIjpudWxsfQ.OFBijJFsVm4IombW6Md1RUsN5v_btPwl-qtY-QSTBQ0b7N2pca8BnnT4OjfXOVRrKCWaKUM3QsGj8zqxCD4xJg 
```

## Reauthentication

If you do not set a value for the `expires` key in the configuration file then your JWT tokens will not expire and will be valid forever.

If you do set an expiration time in your configuration file then the JWT token will expire after that amount of minutes. If the token expires, you will need to reauthenticate to get a new token. This can be done easily by sending the old token to get back a new one:

You can do this by sending a POST request to `/api/reauth` with a `token` input containing the current JWT token. This will check the table for the token and if found, will generate a new token.

## Versions

One of the issues with JWT tokens is there is little that can be done to invalidate JWT tokens. Once a JWT token is valid, it is typically valid forever.

One way to invalid JWT tokens, and force users to reauthenticate, is to specify a version. A JWT token authenticated will contain a version number if one exists. When the JWT token is validated, the version number in the token will check against the version number in the configuration file. If they do not match then the token is considered invalid and the user will have to reauthenticate to get back a new token.

# Loading Users

Since we store the active api_token on the table we are able to retrieve the user using the `LoadUserMiddleware` and a new `guard` route middleware stack:

First add the guard middleware stack and add the `LoadUserMiddleware` to the `api` stack. 

```python
##.. 
"api": [
    JWTAuthenticationMiddleware,
    LoadUserMiddleware
],
"guard": [
    GuardMiddleware
],
```

Lastly, in your route or route groups you can specify the guard middleware and specify the guard name:

```python
# api.py
ROUTES = [
    #..
    Route.get('/users', 'UserController@show').middleware(["guard:jwt"])
]
```
