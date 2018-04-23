# Middleware

## Introduction

Middleware is an extremely important aspect of web applications as it allows you to run important code either before or after every request or even before or after certain routes. In this documentation we'll talk about how middleware works, how to consume middleware and how to create your own middleware. Middleware is only ran when the route is found and a status code of 200 will be returned.

## Getting Started

Middleware classes are placed inside the `app/http/middleware` by convention but can be placed anywhere you like. All middleware are just classes that contain a `before` method and/or an `after` method.

There are four types of middleware in total:

* Middleware ran before every request
* Middleware ran after every request
* Middleware ran before certain routes
* Middleware ran after certain routes

## Creating Middleware

Again, middleware should live inside the `app/http/middleware` folder and should look something like:

```python
class AuthenticationMiddleware:
    ''' Middleware class '''

    def __init__(self, Request):
        self.request = Request

    def before(self):
        pass

    def after(self):
        pass
```

Middleware constructors are resolved by the container so simply pass in whatever you like in the parameter list and it will be injected for you. 

{% hint style="success" %}
Read more about this in the [Service Container](../architectural-concepts/service-container.md) documentation.
{% endhint %}

If Masonite is running a “before” middleware, that is middleware that should be ran before the request, Masonite will check all middleware and look for a `before` method and execute that. The same for “after” middleware.

You may exclude either method if you do not wish for that middleware to run before or after.

This is a boilerplate for middleware. It's simply a class with a before and/or after method. Creating a middleware is that simple. Let's create a middleware that checks if the user is authenticated and redirect to the login page if they are not. Because we have access to the request object from the Service Container, we can do something like:

```python
class AuthenticationMiddleware:
    ''' Middleware class which loads the current user into the request '''

    def __init__(self, Request):
        self.request = Request

    def before(self):
        if not self.request.user():
            self.request.redirectTo('login')

    def after(self):
        pass
```

That's it! Now we just have to make sure our route picks this up. If we wanted this to execute after a request, we could use the exact same logic in the `after` method instead.

Since we are not utilizing the `after` method, we may exclude it all together. Masonite will check if the method exists before executing it.

## Configuration

We have one of two configuration constants we need to work with. These constants both reside in our `config/middleware.py` file and are `HTTP_MIDDLEWARE` and `ROUTE_MIDDLEWARE`.

`HTTP_MIDDLEWARE` is a simple list which should contain an aggregation of your middleware classes. This constant is a list because all middleware will simply run in succession one after another, similar to Django middleware

Middleware is a string to the module location of your middleware class. If your class is located in `app/http/middleware/DashboardMiddleware.py` then the value we place in our middleware configuration will be a string: `app.http.middleware.DashboardMiddleware.DashboardMiddleware`. Masonite will locate the class and execute either the `before` method or the `after` method.

In our `config/middleware.py` file this type of middleware may look something like:

```python
HTTP_MIDDLEWARE = [
    'app.http.middleware.DashboardMiddleware.DashboardMiddleware'
]
```

`ROUTE_MIDDLEWARE` is also simple but instead of a list, it is a dictionary with a custom name as the key and the middleware class as the value. This is so we can specify the middleware based on the key in our routes file.

In our `config/middleware.py` file this might look something like:

```python
from app.http.middleware.RouteMiddleware import RouteMiddleware

ROUTE_MIDDLEWARE = {
    'auth': 'app.http.middleware.RouteMiddleware.RouteMiddleware'
}
```

## Consuming Middleware

Using middleware is also simple. If we put our middleware in the `HTTP_MIDDLEWARE` constant then we don't have to worry about it anymore. It will run on every successful request, that is when a route match is found from our `web.py` file.

If we are using a route middleware, we'll need to specify which route should execute the middleware. To specify which route we can just append a `.middleware()` method onto our routes. This will look something like:

```python
Get().route('/dashboard', 'DashboardController@show').name('dashboard').middleware('auth')
Get().route('/login', 'LoginController@show').name('login')
```

This will execute the auth middleware only when the user visits the `/dashboard` url and as per our middleware will be redirected to the named route of `login`

Awesome! You’re now an expert at how middleware works with Masonite.

