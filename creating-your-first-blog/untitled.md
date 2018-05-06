---
description: >-
  In this part we will walk though the basics of setting up the routes we will
  need to create our blog application.
---

# Part 1 - Creating Our First Route

## Getting Started

All routes are located in `routes/web.py` and are extremely simple to understand. They consist of a request method and a route method. For example, to create a `GET` request it will look like:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/url', 'Controller@method')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We'll talk about the controller in a little bit.

{% hint style="success" %}
You can read more about routes in the [Routing](../the-basics/routing.md) documentation
{% endhint %}

## Creating our Route:

We will start off by creating a view and controller to create a blog post.

A controller is a simple class that holds controller methods. These controller methods will be what our routes will call so they will contain all of our application logic.

{% hint style="info" %}
Think of a controller method as a function in the `views.py` file if you are coming from the Django framework
{% endhint %}

Let's create our first route now. We can put all routes inside `routes/web.py` and inside the `ROUTES` list. You'll see we have a route for the home page. Let's add a route for creating blogs.

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
ROUTES = [
    Get().route('/', 'WelcomeController@show').name('welcome'),
    
    # Blog
    Get().route('/blog', 'BlogController@show')
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You'll notice here we have a `BlogController@show` string. This means "use the blog controller's show method to render this route". The only problem here is that we don't yet have a blog controller.

{% hint style="success" %}
Let's create the `BlogController` in the next step: [Part 2 - Creating Our First Controller](2.-controllers.md)
{% endhint %}



