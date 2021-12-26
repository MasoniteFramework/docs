Adding API Support to your Masonite project is very simple. Masonite comes with supporting for returning JSON responses already but there are a few API features for authenticating and authorizing that are helpful.

Default projects don't come with these features so you'll have to simply register them.

# Getting Started

## Provider

First, register the APIProvider in your project by adding the service provider to your PROVIDERS list:

```python
from masonite.api.providers import APIProvider

PROVIDERS = [
    #..
    APIProvider
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
