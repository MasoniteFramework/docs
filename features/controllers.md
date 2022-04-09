Controllers are a place where most of your business logic will be. Controllers are where you put the responses you see in the web browser. Responses can be dictionaries, lists, views or any class that can render a response.

# Introduction

You may use a craft command to create a new basic controller or simply make a new controller manually. Controllers are classes with methods that are mapped to a route.

Your route may look something like this:

```python
Route.get('/', 'WelcomeController@show')
```

In this case this route will call the `WelcomeController` classes `show` method.

To create a basic controller via a craft command simply run:

```
$ python craft controller Welcome
```

This will create a new controller class to get you setup quickly. This controller class will look like a basic class like this:

```python
from masonite.controllers import Controller
from masonite.views import View


class WelcomeController(Controller):
    def show(self, view: View):
        return view.render("")
```

You may start building your controller out and adding the responses you need.

> Note that the controllers inherit Masonite base `Controller` class. This is required for Masonite to pick up your controller class in routes.

## Dependency Injection

Your controller's constructor and controller methods are resolved by Masonite's [Service Container](../architecture/service-container.md). Because of this, you can simply typehint most of Masonite's classes in either the constructor or the methods:

```python
def __init__(self, request: Request):
  self.request = request

  #..

def show(self, request: Request):
  return request.param('1')
```

Read more about the benefits of the [Service Container](../architecture/service-container.md).

## Responses

Controllers can have different response types based on what you need to return.

### JSON

If you want to return a JSON response you can return a dictionary or a list:

```python
def show(self):
  return {"key": "value"}
```

This will return an `application/json` response.

### Strings

You can return strings:

```python
def show(self):
  return "welcome"
```

### Views

If you want to return a view you can resolve the view class and use the render method:

```python
def show(self, view: View):
  return view.render("views.welcome")
```

### Models

If you are using Masonite ORM, you can return a model directly:

```python
from app.User import User
#..

def show(self, response: Response):
  return User.find(1)
```

### Redirects

If you want to return a redirect you can resolve the response class and use the redirect method:

```python
def show(self, response: Response):
  return response.redirect('/home')
```

### Other

You can return any class that contains a `get_response()` method. This method needs to return any one of the above response types.

## Request Parameters

If you had a parameter in your route, you may fetch it by specifying the parameter in the controller's method:

```python
Route.get('/users/@user_id', 'UsersController@user')
```

Since the `id` parameter is in the route we can fetch the value of this parameter by specifying it in the controller method signature:

```python
def show(self, user_id):
  return User.find(user_id)
```

Another way to fetch route parameters is through the request class:

```python
from masonite.request import Request

def show(self, request: Request):
  return User.find(request.param('user_id'))
```

