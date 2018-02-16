# Request Lifecycle

# Introduction

It's important to know the life cycle of the request so you can understand what is going on under the hood in order to write better software. Whenever you have a better understanding of how your development tools work, you will feel more confident as you'll understand what exactly is going on. This documentation will try to explain in as much detail as needed the flow of the request from initiation to execution. To read about how to use the `Request()` class, read the [Requests](/requests.md) documentation.

# Lifecycle Overview

## First Things

The `Request()` class is in the Masonite package module and is imported and initialized in `bootstrap/start.py` . The `Request()`class is initialized at the top of the WSGI server and takes the WSGI environ variable as a dependency. The Request\(\) class will then parse that environ variable into a usable object for Masonite and pass that around down through the frameworks middleware and controllers. If the route to be executed is an API route, Masonite will load the request into the API class so that it can parse the URI and determine which CRUD operation to perform. After all the middleware, controllers and routes have touched the request object, Masonite will determine if the user, at any point, wants to redirect. See the [Requests](/requests.md) documentation about how redirecting works.

If the request object will be redirecting to a named route, it will cycle through all the routes, locate the route with the name specified, and execute that route. If the request object will be redirecting to a url, it will simply throw a 302 response and send the URI as the Location header on the response. If the user is not being redirected, meaning the redirect method has never been called on the request object, then the response will be a 200 and the data will show as normal.

## Initialization

The request object is initialized at the top of the WSGI server and simply takes the environ variable, which is passed with all WSGI servers and hold information about: the request, server, URI, request method etc. The request object will then parse the environ and store them into certain class attributes. The request object has several methods that assist in extracting these attributes into usable and memorable methods. Read more about how to use the request object in the "Requests" documentation. 

## Requests in Middleware

The request object is sent into middleware and is subject to change there. Any changes to the request object in middleware classes will change the request object for the rest of there workflow. For example, Masonite comes with a `LoadUser` middleware which checks the current user and loads them into the request. This is commented out by default since not all projects need to connect to a database.

If one of the middleware has instructed the request object to redirect, the view that is ready to execute, will not execute.

For example, if the user is planned on going to the dashboard view, but middleware has told the request to redirect to the login page instead, the dashboard view, and therefore the controller, will not execute at all. It will be skipped over. Masonite checks if the request is redirecting before executing a view.

Also, the request object is passed into middleware classes in the constructor so all middleware should load it into the class. Read more about how middleware works in the [Middleware](/middleware.md) documentation.

## Requests in the API

Masonite comes with a built in API system for testing and personal development. It is not yet ready for production and lacks several authentication features.

The request object that is loaded in the API class will check the base URI and then execute CRUD operations based on the URI. As a developer, you are not responsible for manipulating the request object in your API classes.

## Requests and Redirecting

Towards the end of the request lifecycle, Masonite will check if Masonite should redirect the user. This happens whenever the redirecting methods are exectuted on the request object. If the user wants to redirect to a named route, Masonite will loop through all the routes inside `routes/web.py` and check all the route names. If a name matches, it will execute that route. If the route requires parameters \(such that a route has a `@variable` in the route URI\) then you will need to pass the `.send()` method when redirecting. More about request redirecting in the [Requests](/requests.md) documentation.

If the user is redirecting to a normal URL, it will not check any routes. The request will then send a 302 response and redirect the user to the route specified.

## Not Redirecting

If the request is not redirecting, then it will return a 200 status code response and simply show the output needed. This output will come from the controller.

Finally, the data returned from the controller is then passed into the return statement so the WSGI server can interpret it.