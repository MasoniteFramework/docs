# Routing

# Introduction
Masonite Routing is an extremely simple but powerful routing system that at a minimum takes a url and a controller. Masonite will take this route and match it against the requested route and execute the controller on a match.

All routes are created inside `routes/web.py` and are contained in a `ROUTES` constant.  All routes consist of either a `Get()` route or a `Post()` route. At the bare minimum, a route will look like:

```python
Get().route('/url/here', 'WelcomeController@show')
```

Most of your routes will consist of a structure like this. All URI’s should have a preceding `/`. Routes that should only be executed on Post requests (like a form submission) will look very similar:

```python
Post().route('/url/here', 'WelcomeController@store')
```

Notice the controller here is a string. This is a great way to specify controllers as you do not have to import anything into your `web.py` file. All imports will be done in the backend. More on controllers later.

If you wish to not use string controllers and wish to instead import your controller then you can do so by specifying the controller as well as well as only passing a reference to the method.  This will look like:

```python
from app.http.controllers.DashboardController import DashboardController

ROUTES = [
Get().route('/url/here', DashboardController().show)
]
```

It’s important here to recognize that we initialized the controller but only passed a reference to the method and did not actually call the method. This is so Masonite can pass parameters into the method when it executes the route.

## Route Options
There are a few methods you can use to enhance your routes. Masonite typically uses a setters approach to building instead of a parameter approach so to add functionality, we can simply attach more methods.

### Named Routes

We can name our routes so we can utilize these names later when or if we choose to redirect to them. We can specify a route name like so:

```python
Get().route('/dashboard', 'DashboardController@show').name('dashboard')
```

It is good convention to name your routes since route url’s can change but the name should always stay the same.

### Route Middleware

Middleware is a great way to execute classes, tasks or actions either before or after requests. We can specify middleware specific to a route after we have registered it in our `config/middleware.py` file but we can go more in detail in the middleware documentation. To add route middleware we can use the middleware method like so:

```python
Get().route('/dashboard', 'DashboardController@show').middleware('auth', 'anothermiddleware')
```

This middleware will execute either before or after the route is executed depending on the middleware.

### Module Controllers

All controllers are located in `app/http/controllers` but sometimes you may wish to put your controllers in different modules inside the controllers directory. For example, you may wish to put all your product controllers in `app/http/controllers/products` or all of your dashboard controllers in `app/http/controllers/users`. In order to access these controllers in your routes we can simply specify the controller uses our usual dot notation:

```python
Get().route('/dashboard', 'users.DashboardController@show')
```

## Route Parameters
Very often you’ll need to specify parameters in your route in order to retrieve information from your URI. These parameters could be an `id` to use in retrieving a certain model. Specifying route parameters in Masonite is very easy and simply looks like:

```python
Get().route('/dashboard/@id', 'Controller@show')
```

That’s it. This will create a dictionary inside the `Request` object which can be found inside our controllers. 

In order to retrieve our parameters from the request we can use the `param` method on the `Request` object like so:

```python
def show(self, request):
    request.param('id')
```

### Route Parameter Options

Sometimes you will want to make sure that the route parameter is of a certain type. For example you may want to match a URI like `/dashboard/1` but not `/dashboard/joseph`. In order to do this we simply need to pass a type to our parameter. If we do not specify a type then our parameter will default to matching all alphanumeric and underscore characters.

```python
Get().route('/dashboard/@id:int', 'Controller@show')
```

This will match all integers but not strings. So for example will match `/dashboard/10283` and not `/dashboard/joseph`

If we want to match all strings but not integers we can pass .

```python
Get().route('/dashboard/@id:string', 'Controller@show')
```

This will match `/dashboard/joseph` and not `/dashboard/128372`. Currently only the integer and string types are supported.








