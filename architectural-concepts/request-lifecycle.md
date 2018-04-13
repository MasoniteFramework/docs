# Request Lifecycle

## Introduction

It's important to know the life cycle of the request so you can understand what is going on under the hood in order to write better software. Whenever you have a better understanding of how your development tools work, you will feel more confident as you'll understand what exactly is going on. This documentation will try to explain in as much detail as needed the flow of the request from initiation to execution. To read about how to use the `Request` class, read the [Requests](../the-basics/requests.md) documentation.

## Lifecycle Overview

### First Things

All of Masonite is bootstrapped via [Service Providers](service-providers.md) and is centralized via a [Service Container](service-container.md). The `Request` class in binded into the container inside the `AppProvider` Service Provider which is inside the Masonite pip package. The `Request` class is initialized and registered into the container before the WSGI server starts. Because of this, the `Request` class is the same instance throughout the life of the server. In other words, the `Request` class is a singleton for the life of the WSGI server and is not re-instantiated on each request. On each request, the WSGI server will load the `environ` variable \(the WSGI request details\) into the request class which changes the behavior of requests accordingly such as what the URI is or what the request method type is.

### Initialization

The request object is initialized in the `AppProvider` Service Provider and simply takes the environ variable as a dependency which is passed with all WSGI servers and hold information about the: request, server, URI, request method etc. The request object will then parse the environ and store them into certain class attributes. The request object has several methods that assist in extracting these attributes into usable and memorable methods. Read more about how to use the request object in the [Requests](../the-basics/requests.md) documentation.

### Requests in Middleware

If one of the middleware has instructed the request object to redirect, the view that is ready to execute, will not execute.

For example, if the user is planned on going to the dashboard view, but middleware has told the request to redirect to the login page instead then the dashboard view, and therefore the controller, will not execute at all. It will be skipped over. Masonite checks if the request is redirecting before executing a view.

Also, the request object can be injected into middleware by passing the `Request` parameter into the constructor like so:

```python
class SomeMiddleware:

    def __init__(self, Request):
        self.request = Request

    ...
```

This will inject the `Request` class into the constructor when that middleware is executed. Read more about how middleware works in the [Middleware](../advanced/middleware.md) documentation.

### Requests in the API

Masonite comes with a built in API system for testing and personal development. It is not yet ready for production and lacks several authentication features.

The request object that is loaded in the API class will check the base URI and then execute CRUD operations based on the URI. As a developer, you are not responsible for manipulating the request object in your API classes.

### Requests and Redirecting

Towards the end of the request lifecycle, Masonite will check if Masonite should redirect the user. This happens whenever the redirecting methods are exectuted on the request object. If the user wants to redirect to a named route, Masonite will loop through all the routes inside `routes/web.py` and check all the route names. If a name matches, it will execute that route. If the route requires parameters \(such that a route has a `@variable` in the route URI\) then you will need to pass the `.send()` method when redirecting. More about request redirecting in the [Requests](../the-basics/requests.md) documentation.

If the user is redirecting to a normal URL, it will not check any routes. The request will then send a 302 response and redirect the user to the route specified.

### Not Redirecting

If the request is not redirecting, then it will return a 200 status code response and simply show the output needed. This output will come from the controller.

Finally, the data returned from the controller is then passed into the return statement so the WSGI server can interpret it.

