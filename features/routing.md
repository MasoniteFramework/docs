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

## Available Route Options

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

## Middleware

You may define several available methods on your routes to modify their behavior during the request.

The first option is to define middleware:

```python
Route.get('/welcome', 'WelcomeController@show').middleware('web')
```

This will attach the middleware key to the route which will be picked up from your middleware configuration later in the request.

## Name

You can specify a name for your route. This is used to easily compile route information in other parts of your application by using the route name which is much more static than a URL.

```python
Route.get('/welcome', 'WelcomeController@show').name('welcome')
```

## Parameters

You can specify the parameters in the URL that will later be able to be retrieved in other parts of your application. You can do this easily by specify the parameter name attached to a `@` symbol:

```python
Route.get('/dashboard/@user_id', 'WelcomeController@show').name('welcome')
```

## Domain

You can specify the subdomain you want this route to be matched to. If you only want this route to be matched on a "docs" subdomain (docs.example.com):

```python
Route.get('/dashboard/@user_id', 'WelcomeController@show').domain('docs')
```

## Route Compilers

Route compilers are a way to match on a certain route parameter by a specific type. For example, if you only watch to match where the `@user_id` is an integer. You can do this by appending a `:` character and compiler name to the parameter:

```python
Route.get('/dashboard/@user_id:string', 'WelcomeController@show').name('welcome')
```

Available route compilers are:

* `integer`
* `int` (alias for integer)
* `string`
* `signed`

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

