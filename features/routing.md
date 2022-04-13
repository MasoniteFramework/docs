Masonite ships with a really powerful routing engine. Routing helps link a browser URL to its controller and controller action.

# Creating a Route

Routes are created by importing the `Route` class and defining the HTTP verb you would like with the URL and the controller you would like to use. These routes need to be wrapped in a `ROUTES` list inside your routes file.

```python
from masonite.routes import Route

ROUTES = [
	Route.get('/welcome', 'WelcomeController@show')
]
```

The first parameter is the URL you would like to be available in your application. In the above example, this will allow anybody to go to the `/welcome` URL.

## Available Route Methods

You may choose to define any one of the available verbs:

```python
Route.get('/welcome', 'WelcomeController@show')
Route.post('/welcome', 'WelcomeController@show')
Route.put('/welcome', 'WelcomeController@show')
Route.patch('/welcome', 'WelcomeController@show')
Route.delete('/welcome', 'WelcomeController@show')
Route.options('/welcome', 'WelcomeController@show')
```

In addition to these route verbs you can use built in routes:

```python
Route.redirect('/old', '/new', status=301)
Route.permanent_redirect('/old', '/new')
```

# Route Options

You may define several available methods on your routes to modify their behavior during the request.

## Middlewares

You can add one or multiple [Routes Middlewares](/features/middleware.md#route-middleware):

```python
Route.get('/welcome', 'WelcomeController@show').middleware('web')
Route.get('/settings', 'WelcomeController@settings').middleware('auth', 'web')
```

This will attach the middleware key(s) to the route which will be picked up from your middleware configuration later in the request.

You can exclude one or multiple [Routes Middlewares](/features/middleware.md#route-middleware) for a specific route:

```python
Route.get('/about', 'WelcomeController@about').exclude_middleware('auth', 'custom')
```

## Name

You can specify a name for your route. This is used to easily compile route information in other parts of your application by using the route name which is much more static than a URL.

```python
Route.get('/welcome', 'WelcomeController@show').name('welcome')
```

## Parameters

You can specify the parameters in the URL that will later be able to be retrieved in other parts of your application. You can do this easily by specify the parameter name attached to a `@` symbol:

```python
Route.get('/dashboard/@user_id', 'WelcomeController@show')
```

## Optional Parameters

Sometimes you want to optionally match routes and route parameters. For example you may want to match `/dashboard/user` and `/dashboard/user/settings` to the same controller method. In this event you can use optional parameters which are simply replacing the `@` symbol with a `?`:

```python
Route.get('/dashboard/?option', 'WelcomeController@show')
```

## Domain

You can specify the subdomain you want this route to be matched to. If you only want this route to be matched on a "docs" subdomain (docs.example.com):

```python
Route.get('/dashboard/@user_id', 'WelcomeController@show').domain('docs')
```

## Route Compilers

Route compilers are a way to match on a certain route parameter by a specific type. For example, if you only watch to match where the `@user_id` is an integer. You can do this by appending a `:` character and compiler name to the parameter:

```python
Route.get('/dashboard/@user_id:string', 'WelcomeController@show')
```

Available route compilers are:

* `integer`
* `int` (alias for integer)
* `string`
* `signed`

## Creating Route Compilers

You can also create your own route compilers if you want to be able to support specific regex matches for routes.

All route compilers will need to be added to the top of your `register_routes()` method in your `Kernel.py` file.

```python
    def register_routes(self):
        Route.set_controller_locations(self.application.make("controllers.location"))
        Route.compile("handle", r"([\@\w\-=]+)")

        #..
```

> Note: The compile methods need to happen before the routes are loaded in this method. You may also put it in any method that appears before the `register_routes()` method.

# Route Groups

Route groups are a great way to group mulitple routes together that have similiar options like a prefix, or multiple routes with the same middleware.

A route uses the `group()` method that accepts a list of routes and keyword arguments for the options:

```python
ROUTES = [
  Route.group([
    Route.get('/settings', 'DashboardController@settings').name('settings'),
    Route.get('/monitor', 'DashboardController@monitor').name('monitor'),
  ],
  prefix="/dashboard",
  middleware=['web', 'cors'],
  name="dashboard."),
  domain="docs"
]
```

The prefix and name options will prefix the options set in the routes inside the group. In the above example, the names of the routes would `dashboard.settings` with a URL of `/dashboard/settings` and `dashboard.monitor` and a URL of `/dashboard/monitor`.

# List Routes

Application routes can be listed with the `routes:list` Masonite command. Routes will be displayed
in a table with relevant info such as route name, methods, controller and enabled middlewares for this route.

Routes can be filtered by methods:
```bash
python craft routes:list -M POST,PUT
```

Routes can be filtered by name:

```bash
python craft routes:list -N users
```
