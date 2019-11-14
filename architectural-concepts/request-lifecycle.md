# Request Lifecycle

## Introduction

It's important to know the life cycle of the request so you can understand what is going on under the hood in order to write better software. Whenever you have a better understanding of how your development tools work, you will feel more confident as you'll understand what exactly is going on. This documentation will try to explain in as much detail as needed the flow of the request from initiation to execution. To read about how to use the `Request` class, read the [Requests](../the-basics/requests.md) documentation.

## Lifecycle Overview

Masonite is bootstrapped via [Service Providers](service-providers.md), these providers load features, hooks, drivers and other objects into the [Service Container](service-container.md) which can then be pulled out by you, the developer, and used in your views, middleware and drivers.

With that being said, not all Service Providers need to be ran on every request and there are good times to load objects into the container. For example, loading routes into the container does not need to be ran on every request. Mainly because they won't change before the server is restarted again.

Now the entry point when the server is first ran \(with something like `craft serve`\) is the `wsgi.py` file in the root of the directory. In this directory, all Service Providers are registered. This means that objects are loaded into the container first that typically need to be used by any or all Service Providers later. All Service Providers are registered regardless on whether they require the server to be running \(more on this later\).

Now it's important to note that the server is not yet running we are still in the `wsgi.py` file but we have only hit this file, created the container, and registered our Service Providers.

Right after we register all of our Service Providers, we break apart the provider list into two separate lists. The first list is called the `WSGIProviders` list which are providers where `wsgi=True` \(which they are by default\). We will use this list of a smaller amount of providers in order to speed up the application since now we won't need to run through all providers and see which ones need to run.

While we are in the loop we also create a list of providers where `wsgi=False` and boot those providers. These boot methods may contain things like Manager classes creating drivers which require all drivers to be registered first but doesn't require the WSGI server to be running.

Also, more importantly, the WSGI key is binded into the container at this time. The default behavior is to wrap the WSGI application in a Whitenoise container to assist in the straight forwardness of static files.

{% hint style="info" %}
This behavior can be changed by swapping that Service Provider with a different one if you do not want to use Whitenoise.
{% endhint %}

Once all the register methods are ran and all the boot methods of Service Providers where wsgi is false, and we have a WSGI key in the container, we can startup the server by using the value of the WSGI key.

We then make an instance of the WSGI key from the container and set it to an application variable in order to better show what is happening. Then this is where the WSGI server is started.

## The Server

Now that we have the server running, we have a new entry point for our requests. This entry point is the app function inside bootstrap/start.py.

Now all wsgi servers set a variable called environ. In order for our Service Providers to handle this, we bind it into the container to the Environ key.

Next we run all of our Service Providers where wsgi is true now \(because the WSGI server is running\).

## WSGI Service Providers

The Request Life Cycle is now going to hit all of these providers. Although you can obviously add any Service Providers you at any point in the request, Masonite comes with 5 providers that should remain in the order they are in. These providers have been commented as `# Framework Providers`. Because the request needs to hit each of these in succession, they should be in order although you may put any amount of any kind of Service Providers in between them.

We enter into a loop with these 5 Service Providers and they are the:

### App Provider

This Service Provider registered objects like the routes, request class, response, status codes etc into the container and then loads the environment into these classes on every request so that they can change with the environment. Remember that we registered these classes when the server first started booting up so they remain the same class object as essentially act as singletons Although they aren't being reinstantiated with the already existing object, they are instantiated once and die when the server is killed.

### Session Provider

If one of the middleware has instructed the request object to redirect, the view that is ready to execute, will not execute.

For example, if the user is planned on going to the dashboard view, but middleware has told the request to redirect to the login page instead then the dashboard view, and therefore the controller, will not execute at all. It will be skipped over. Masonite checks if the request is redirecting before executing a view.

Also, the request object can be injected into middleware by passing the `Request` parameter into the constructor like so:

```python
from masonite.request import Request

class SomeMiddleware:

    def __init__(self, request: Request):
        self.request = request

    ...
```

This will inject the `Request` class into the constructor when that middleware is executed. Read more about how middleware works in the [Middleware](../advanced/middleware.md) documentation.

This provider loads the ability to use sessions, adds a session helper to all views and even attaches a session attribute to the request class.

### Route Provider

This provider takes the routes that are loaded in and makes the response object, static codes, runs middleware and other route displaying specific logic. This is the largest Service Provider with the most logic. This provider also searches through the routes, finds which one to hit and exectues the controller and controller method.

### Status Code Provider

This provider is responsible for showing the nice HTTP status codes you see during development and production. This Service Provider also allows custom HTTP error pages by putting them into the `resources/templates/errors` directory.

Nothing too special about this Service Provide. You can remove this if you want it to show the default WSGI server error.

### Start Response

This Service Provider collects a few classes that have been manipulated by the above Service Providers and constructs a few headers such as the content type, status code, setting cookies and location if redirecting.

## Leaving the Providers

Once these 5 providers have been hit \(and any you add\), we have enough information to show the output. We leave the Service Provider loop and set the response and output which are specific to WSGI. The output is then sent to the browser with any cookies to set, any new headers, the response, status code, and everything else you need to display html \(or json\) to the browser.

