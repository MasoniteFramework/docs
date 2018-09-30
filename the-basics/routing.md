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
    Get().route('/url/here', DashboardController.show)
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
It’s important here to recognize that we didn't initialize the controller or the method, we did not actually call the method. This is so Masonite can pass parameters into the constructor and method when it executes the route, typically through auto resolving dependency injection.
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

Most developers choose to use these instead of the classes.

### Route Groups

Some routes may be very similar. We may have a group of routes under the same domain, uses the same middleware or even start with the same prefixes. In these instances we should group our routes together so they are more DRY and maintainable.

We can add route groups like so:

```python
from masonite.routes import RouteGroup
from masonite.helpers.routes import get

ROUTES = [

    RouteGroup([
        get('/url1', ...),
        get('/url2', ...),
        get('/url3', ...),
    ]),

]
```

This alone is great to group routes together that are similar but in addition to this we can add specific attributes to the entire group like adding middleware:

```python
ROUTES = [

    RouteGroup([
        get('/url1', ...),
        get('/url2', ...),
        get('/url3', ...),
    ], middleware=('auth', 'jwt')),

]
```

In this instance we are adding these 2 middleware to all of the routes inside the group. We have access to a couple of different methods. Feel free to use some or all of these options:

```python
ROUTES = [

    RouteGroup([
        get('/url1', ...).name('create'),
        get('/url2', ...).name('update'),
        get('/url3', ...).name('delete'),
    ], 
    middleware=('auth', 'jwt'),
    domain='subdomain',
    prefix='/dashboard',
    name='post.'
    ),

]
```

The `prefix` parameter will prefix that URL to all routes in the group as well as the `name` parameter. The code above will create routes like `/dashboard/url1` with the name of `post.create`. As well as adding the domain and middleware to the routes.

All of the options in a route group are named parameters so if you think adding a groups attribute at the end is weird you can specify them in the beginning and add the `routes` parameter:

```python
RouteGroup(middleware=('auth', 'jwt'), name='post.', routes = [
    get('/url1', ...).name('create'),
    get('/url2', ...).name('update'),
    get('/url3', ...).name('delete'),
]),
```

### Multiple Route Groups

Even more awesome is the ability to nest route groups:

```python
ROUTES = [

    RouteGroup([
        get('/url1', ...).name('create'),
        get('/url2', ...).name('update'),
        get('/url3', ...).name('delete'),
        RouteGroup([
            get('/url4', ...).name('read'),
            get('/url5', ...).name('put'),
        ], prefix='/users', name='user.'),
    ], prefix='/dashboard', name='post.', middleware=('auth', 'jwt')),

]
```

This will go to each layer and generate a route list essentially from the inside out. For a real world example we refactor routes from this:

```python
ROUTES = [
    Get().domain('www').route('/', 'WelcomeController@show').name('welcome'),
    Post().domain('www').route('/invite', 'InvitationController@send').name('invite'),
    Get().domain('www').route('/dashboard/apps', 'AppController@show').name('app.show').middleware('auth'),
    Get().domain('www').route('/dashboard/apps/create', 'AppController@create').name('app.create').middleware('auth'),
    Post().domain('www').route('/dashboard/apps/create', 'AppController@store').name('app.store'),
    Post().domain('www').route('/dashboard/apps/delete', 'AppController@delete').name('app.delete'),
    Get().domain('www').route('/dashboard/plans', 'PlanController@show').name('plans').middleware('auth'),
    Post().domain('www').route('/dashboard/plans/subscribe', 'PlanController@subscribe').name('subscribe'),
    Post().domain('www').route('/dashboard/plans/cancel', 'PlanController@cancel').name('cancel'),
    Post().domain('www').route('/dashboard/plans/resume', 'PlanController@resume').name('resume'),

    Post().domain('*').route('/invite', 'InvitationController@subdomain').name('invite.subdomain'),
    Get().domain('*').route('/', 'WelcomeController@subdomain').name('welcome'),
]

ROUTES = ROUTES + [
    Get().domain('www').route('/login', 'LoginController@show').name('login'),
    Get().domain('www').route('/logout', 'LoginController@logout'),
    Post().domain('www').route('/login', 'LoginController@store'),
    Get().domain('www').route('/register', 'RegisterController@show'),
    Post().domain('www').route('/register', 'RegisterController@store'),
    Get().domain('www').route('/home', 'HomeController@show').name('home'),
]
```

into this:

```python
ROUTES = [

    RouteGroup([
        # Dashboard Routes
        RouteGroup([
            # App Routes
            RouteGroup([
                Get().route('', 'AppController@show').name('show'),
                Get().route('/create', 'AppController@create').name('create'),
                Post().route('/create', 'AppController@store').name('store'),
                Post().route('/delete', 'AppController@delete').name('delete'),
            ], prefix='/apps', name='app.'),

            Get().route('/plans', 'PlanController@show').name('plans'),
            Post().route('/plans/subscribe', 'PlanController@subscribe').name('subscribe'),
            Post().route('/plans/cancel', 'PlanController@cancel').name('cancel'),
            Post().route('/plans/resume', 'PlanController@resume').name('resume'),
        ], prefix="/dashboard", middleware=('auth',)),

        # Login and Register Routes
        Get().route('/login', 'LoginController@show').name('login'),
        Get().route('/logout', 'LoginController@logout'),
        Post().route('/login', 'LoginController@store'),
        Get().route('/register', 'RegisterController@show'),
        Post().route('/register', 'RegisterController@store'),
        Get().route('/home', 'HomeController@show').name('home'),

        # Base Routes
        Get().route('/', 'WelcomeController@show').name('welcome'),
        Post().route('/invite', 'InvitationController@send').name('invite'),
    ], domain='www'),


    # Subdomain invitation routes
    Post().domain('*').route('/invite', 'InvitationController@subdomain').name('invite.subdomain'),
    Get().domain('*').route('/', 'WelcomeController@subdomain').name('welcome'),
]
```

This will likely be the most common way to build routes for your application.

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

### Global Controllers

Controllers are defaulted to the `app/http/controllers` directory but you may wish to completely change the directory for a certain route. We can use a forward slash in the beginning of the controller namespace:

```python
Get().route('/dashboard', '/thirdparty.package.users.DashboardController@show')
```

This can enable us to use controllers in third party packages.

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

## Route Compilers

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

{% hint style="info" %}
These are called "Route Compilers" because they compile the route differently depending on what is specified. If you specify `:int` or `:integer` it will compile to a different regex than if you specified `:string`.
{% endhint %}

### Adding Route Compilers

We can add route compilers to our project by specifying them in a Service Provider.

Make sure you add them in a Service Provider where `wsgi` is `False`. We can add them on the Route class from the container using the `compile` method. A completed example might look something like this:

{% code-tabs %}
{% code-tabs-item title="app/http/providers/RouteCompileProvider.py" %}
```python
class RouteCompilerProvider(ServiceProvider):

    wsgi = False
    ...

    def boot(self, Route):
        Route.compile('year', r'([0-9]{4})')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We just need to call the `compile()` method on the `Route` class and make sure we specify a regex string by preceding an `r` to the beginning of the string.

{% hint style="info" %}
Your regex should be encapsulated in a group. If you are not familiar with regex, this basically just means that your regex pattern should be inside parenthesis like the example above.
{% endhint %}

### Subdomain Routing

You may wish to only render routes if they are on a specific subdomain. For example you may want `example.com/dashboard` to route to a different controller than `joseph.example.com/dashboard`.

Out of the box this feature will not work and is turned off by default. We will need to add a call on the Request class in order to activate subdomains. We can do this in the boot method of one of our Service Providers that has wsgi=False:

{% code-tabs %}
{% code-tabs-item title="app/providers/UserModelProvider.py" %}
```python
wsgi = False
...
def boot(self, Request):
    Request.activate_subdomains()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

To use subdomains we can use the `.domain()` method on our routes like so:

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

