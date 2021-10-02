Middleware is an extremely important aspect of web applications as it allows you to run important code either before or after every request or even before or after certain routes. In this documentation we'll talk about how middleware works, how to consume middleware and how to create your own middlewar

## Getting Started

Middleware classes are placed inside the `Kernel` class. All middleware are just classes that contain a `before` method and `after` method.

There are four types of middleware in total:

* Middleware ran before every request
* Middleware ran after every request
* Middleware ran before certain routes
* Middleware ran after certain routes

## Configuration

We have one of two configuration attributes we need to work with. These attributes both reside in our `Kernel` file and are `http_middleware` and `route_middleware`.

`http_middleware` is a simple list which should contain your middleware classes. This attribute is a list because all middleware will simply run in succession one after another, similar to Django middleware

In our `Kernel` file this type of middleware may look something like:

```python
from masonite.middleware import EncryptCookies

class Kernel:

    http_middleware = [
        EncryptCookies
    ]
    
```

Middleware will run on every inbound request to the application whether or not a route was found or not.

## Route Middleware

Route middleware is also simple but instead of a list, it is a dictionary with a custom name as the key and a list of middleware. This is so we can specify the middleware based on the key in our routes file.

In our `config/middleware.py` file this might look something like:

```python
from app.middleware.RouteMiddleware import RouteMiddleware
class Kernel:
  
  #..

  route_middleware = {
      "web": [
          SessionMiddleware, 
          HashIDMiddleware,
          VerifyCsrfToken,
      ],
  }
```

> By default, all routes inside the `web.py` file will run the `web` middleware list

## Middleware Parameters

You can pass parameters from your routes to your middleware in cases where a middleware should act differently depending on your route.

You can do this with a `:` symbol next to your route middleware name and then pass in those parameters to the `before` and `after` middleware methods.

For example, we may be creating a middleware for request throttling and in our routes we have something like this:

```python
Get('/feeds', 'FeedController').middleware('throttle:2,100')
```

notice the throttle:2,100 syntax. The 2 and the 100 will then be passed into the before and after methods of your middleware:

```python
class ThrottleMiddleware:

    def before(self, request, response, minutes, requests):
        # throttle requests

    def after(self, request, response, 
             minutes, requests):
        # throttle requests
```

## Request Parameters

Similiar to the way we can pass values to a middleware using the `:` splice we can also use the `@` in the value to pass the value of the parameter.

For example, we may create a route and a middleware like this

```python
Get('/dashboard/@user_id/settings', 'FeedController').middleware('permission:@user_id')
```

If we go to a route like `/dashboard/152/settings` then the value of 152 will be passed to the middleware before and after methods.

## Creating Middleware

Middleware:

* can live anywhere in your project, 
* Inherit from Masonite's base middleware class
* Contain a before and after method that accepts request and response parameters

```python
from masonite.middleware import Middleware

class AuthenticationMiddleware(Middleware):
    """Middleware class
    """

    def before(self, request, response):
        #..
        return request

    def after(self, request, response):
        #..
        return request
```

**It's important to note that in order for the request lifecycle to continue, you must return the request class. If you do not return the request class, no other middleware will run after that middleware.** 

That's it! Now we just have to make sure our route picks this up. If we wanted this to execute after a request, we could use the exact same logic in the `after` method instead.

## Consuming Middleware

If we are using a route middleware, we'll need to specify which route should execute the middleware. To specify which route we can just append a `.middleware()` method onto our routes. This will look something like:

```python
Route.get('/dashboard', 'DashboardController@show').name('dashboard').middleware('auth')
Route.get('/login', 'LoginController@show').name('login')
```
