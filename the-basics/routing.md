# Routing

## Introduction

Masonite Routing is an extremely simple but powerful routing system that at a minimum takes a url and a controller. Masonite will take this route and match it against the requested route and execute the controller on a match.

All routes are created inside `routes/web.py` and are contained in a `ROUTES` constant. All routes consist of either a `Get()` route or a `Post()` route. At the bare minimum, a route will look like:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/url/here', 'WelcomeController@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Most of your routes will consist of a structure like this. All URI’s should have a preceding `/`. Routes that should only be executed on Post requests \(like a form submission\) will look very similar:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Post().route('/url/here', 'WelcomeController@store')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
Notice the controller here is a string. This is a great way to specify controllers as you do not have to import anything into your `web.py` file. All imports will be done in the backend. More on controllers later.
{% endhint %}

If you wish to not use string controllers and wish to instead import your controller then you can do so by specifying the controller as well as well as only passing a reference to the method. This will look like:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
...
from app.http.controllers.DashboardController import DashboardController


ROUTES = [
    Get().route('/url/here', DashboardController().show)
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
It’s important here to recognize that we initialized the controller but only passed a reference to the method and did not actually call the method. This is so Masonite can pass parameters into the method when it executes the route.
{% endhint %}

## Route Options

There are a few methods you can use to enhance your routes. Masonite typically uses a setters approach to building instead of a parameter approach so to add functionality, we can simply attach more methods.

### HTTP Verbs

There are several HTTP verbs you can use for routes:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
from masonite.routes import Get, Post, Put, Patch, Delete

Get().route(...)
Post().route(...)
Put().route(...)
Patch().route(...)
Delete().route(...)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **HTTP Helpers**

If the syntax is a bit cumbersome, you just want to make it shorter or you like using shorthand helper functions, then you can also use these:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
from masonite.helpers.routes import get, post, put, patch, delete

ROUTES = [
    get('/url/here', 'Controller@method'),
    post('/url/here', 'Controller@method'),
    put('/url/here', 'Controller@method'),
    patch('/url/here', 'Controller@method'),
    delete('/url/here', 'Controller@method'),
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

These return instances of their respective classes so you can append on to them:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
get('/url/here', 'Controller@method').middleware(...),
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Named Routes

We can name our routes so we can utilize these names later when or if we choose to redirect to them. We can specify a route name like so:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/dashboard', 'DashboardController@show').name('dashboard')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

It is good convention to name your routes since route URI's can change but the name should always stay the same.

### Route Middleware

Middleware is a great way to execute classes, tasks or actions either before or after requests. We can specify middleware specific to a route after we have registered it in our `config/middleware.py` file but we can go more in detail in the middleware documentation. To add route middleware we can use the middleware method like so:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/dashboard', 'DashboardController@show').middleware('auth', 'anothermiddleware')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This middleware will execute either before or after the route is executed depending on the middleware.

{% hint style="success" %}
Read more about how to use and create middleware in the [Middleware ](../advanced/middleware.md)documentation.
{% endhint %}

### Deeper Module Controllers

All controllers are located in `app/http/controllers` but sometimes you may wish to put your controllers in different modules **deeper** inside the controllers directory. For example, you may wish to put all your product controllers in `app/http/controllers/products` or all of your dashboard controllers in `app/http/controllers/users`. In order to access these controllers in your routes we can simply specify the controller using our usual dot notation:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/dashboard', 'users.DashboardController@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Change Controller Modules

Controllers are defaulted to the `app/http/controllers` directory but you may wish to completely change the directory for a certain route. We can change this by using the `.module()` method:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().module('thirdparty.package').route('/dashboard', 'users.DashboardController@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will look for the controller in `thirdparty.package.users` module instead of the normal `app.http.controllers` module.

You may also start the controller string with a / which will look for the controller as a global package:

```python
Get().route('/dashboard', '/thirdparty.package.users.DashboardController@show')
```

Which will look for the package in the exact same way as before.

## Route Parameters

Very often you’ll need to specify parameters in your route in order to retrieve information from your URI. These parameters could be an `id` for the use in retrieving a certain model. Specifying route parameters in Masonite is very easy and simply looks like:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/dashboard/@id', 'Controller@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

That’s it. This will create a dictionary inside the `Request` object which can be found inside our controllers.

In order to retrieve our parameters from the request we can use the `param` method on the `Request` object like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controller/YourController.py" %}
```python
def show(self, Request):
    Request.param('id')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Route Parameter Options

Sometimes you will want to make sure that the route parameter is of a certain type. For example you may want to match a URI like `/dashboard/1` but not `/dashboard/joseph`. In order to do this we simply need to pass a type to our parameter. If we do not specify a type then our parameter will default to matching all alphanumeric and underscore characters.

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/dashboard/@id:int', 'Controller@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will match all integers but not strings. So for example it will match `/dashboard/10283` and not `/dashboard/joseph`

If we want to match all strings but not integers we can pass:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/dashboard/@id:string', 'Controller@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will match `/dashboard/joseph` and not `/dashboard/128372`. Currently only the integer and string types are supported.

### Subdomain Routing

You may wish to only render routes if they are on a specific subdomain. For example you may want `example.com/dashboard` to route to a different controller than `joseph.example.com/dashboard`. To do this we can use the `.domain()` method on our routes like so:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().domain('joseph').route('/dashboard', 'Controller@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This route will match to `joseph.example.com/dashboard` but not to `example.com/dashboard` or `test.example.com/dashboard`.

It may be much more common to match to any subdomain. For this we can pass in an asterisk instead.

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().domain('*').route('/dashboard', 'Controller@show')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will match all subdomains such as `test.example.com/dashboard`, `joseph.example.com/dashboard` but not `example.com/dashboard`.

If a match is found, it will also add a `subdomain` parameter to the Request class. We can retrieve the current subdomain like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self, Request):
    print(Request.param('subdomain'))
```
{% endcode-tabs-item %}
{% endcode-tabs %}



