# Optimization

## Introduction

Masonite can handle most of the application that will be built with it. Python is clearly not the fastest language so if it is brute speed to manage 100,000 simultaneous connections than no application written in Python will suit your needs. Those types of application will typically require some Node.js server and asynchronous tasks on multiple servers but if you are going with a Python framework than it is likely you will not need that much juice.

## WSGI Server

Out of the box, Masonite is designed to run as a great development environment. There are a few trade off's that need to be made in order to stably run Masonite on all operating systems. Masonite defaults to running as the WSGI server with Waitress. This was chosen because waitress is a WSGI server that runs on both Windows and Mac and is powerful enough to run most application during development.

As you're application is entering the deployment phase you should look away from running the server with Waitress and look to more production ready options such as uWSGI and Gunicorn. Gunicorn has been tested to run faster than uWSGI both using their default options but uWSGI may be able to be tweaked in a way that is faster dependent on your application.

Because Masonite is simply a WSGI application, any deployment tutorials you research should simply be how to setup a WSGI application on the platform you are trying to deploy to. 

So remember, Waitress is great for development but a better WSGI server like Gunicorn should be used when deploying to production.

## Rearranging Providers

Masonite ships with a single list of providers which is excellent for getting the application done fast and is very simple to understand. 

{% hint style="info" %}
If you have not read how [Service Providers](../architectural-concepts/service-providers.md) work, be sure to read about them in the [Service Provider](../architectural-concepts/service-providers.md) documentation.
{% endhint %}

Masonite will work perfectly fine for most applications and your provider list should go untouched for the most part. If you are craving just a little bit more speed then you should rearrange your provider list to boost the efficiency a bit.

{% hint style="success" %}
Before continuing, it would make sense to read about the [Request Lifecycle](../architectural-concepts/request-lifecycle.md) in order to understand why this section makes the application faster.
{% endhint %}

### Making Two Provider Lists

If you have read the Request Lifecycle and the Service Provider documentation than you may have noticed that there are only a few Service Providers that need to be ran on every request. All the other providers "bootstrap" the container and allow the framework to have features. The problem with this is that since the `PROVIDERS` list is a single list, we have to search through each provider and check if it should run on every request.

A great option is to break up the providers into 2 lists. The first list is a list containing only the providers that need to run on every request. 

{% hint style="info" %}
These providers have the `wsgi = True` by default. If you do not see a `wsgi` attribute on a Service Provider then it is running on every request. You must explicitly set it to `False` to turn that off so it only runs when the server first starts up.
{% endhint %}

So let's do this and break up the PROVIDERS list into two separate lists. Here is an example of what that might look like:

{% code-tabs %}
{% code-tabs-item title="config/providers.py" %}
```python
from masonite.providers import (
    AppProvider,
    SessionProviders,
    ...
    ...
)

WSGI_PROVIDERS = [ 
    AppProvider,
    SessionProvider,
    RouteProvider, 
    StatusCodeProvider, 
    StartResponseProvider, 
]

PROVIDERS = [
    # Framework Providers
    SassProvider,
    WhitenoiseProvider,
    ViewProvider,
    HelpersProvider,

    # Optional Framework Providers
    MailProvider,
    UploadProvider,
    QueueProvider,
    CacheProvider,
    BroadcastProvider,
    CacheProvider,
    CsrfProvider,

    # Third Party Providers
    BillingProvider,
    SentryServiceProvider,
    DashboardProvider,

    # Application Providers
    UserModelProvider,
    MiddlewareProvider,
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now we only have 5 providers that are in a list of their own in which 

