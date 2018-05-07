---
description: >-
  In this part we will add some boilerplate html markup to our application.
  We'll be using a basic Bootstrap 3 template.
---

# Part 6 - Designing Our Blog

## Getting Started

Let's setup a little HTML so we can learn a bit more about how views work. In this part we will setup a really basic template in order to not clog up this part with too much HTML but we will learn the basics enough that you can move forward and create a really awesome blog template \(or collect one from the internet\).

Now that we have all the models and migrations setup, we have everything in the backend that we need to create layout and start creating and updating blog posts.

We will check if the user logged in before creating a template.

## The Template For Creating

The template for creating will be located at blog/create and will be a simple form for creating a blog post

```markup
<form action="/blog/create" method="POST">
    {{ csrf_field|safe }}

    <input type="name" name="title">
    <textarea name="body"></textarea>
</form>
```

Notice here we have this strange {{ csrf\_field\|safe }} looking text. Masonite comes with CSRF protection so we need a token to render with the CSRF field.

Now because we have a foreign key in our posts table, we need to make sure the user is logged in before creating this so let's change up our template a bit:

```markup
{% if auth() %}
    <form action="/blog/create" method="POST">
        {{ csrf_field|safe }}

        <label> Title </label>
        <input type="name" name="title"><br>
        
        <label> Body </label>
        <textarea name="body"></textarea>
        
        <input type="submit" value="Post!">
    </form>
{% else %}
    <a href="/login">Please Login</a>
{% endif %}
```

{% hint style="success" %}
Masonite uses Jinja2 templating so if you don't understand this templating, be sure to [read their documentation](http://jinja.pocoo.org/docs/2.10/).
{% endhint %}

## Static Files

For simplicity sake, we won't by styling our blog with something like Bootstrap but it is important to learn how static files such as CSS files work with Masonite so let's walk through how to add a CSS file and add it to our blog.

Firstly, head to storage/static/ and make a blog.css file and throw anything you like in it. For this tutorial we will make the html page slightly grey.

{% code-tabs %}
{% code-tabs-item title="storage/static/blog.css" %}
```css
html {
    background-color: #ddd;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now we can add it to our template like so:

```markup
<link href="/static/blog.css" rel="stylesheet">
{% if auth() %}
    <form action="/blog/create" method="POST">
        {{ csrf_field|safe }}

        <label> Title </label>
        <input type="name" name="title"><br>
        
        <label> Body </label>
        <textarea name="body"></textarea>
        
        <input type="submit" value="Post!">
    </form>
{% else %}
    <a href="/login">Please Login</a>
{% endif %}
```

That's it. Static files are really simple. It's important to know how they work but for this tutorial we will ignore them for now and focus on more of the backend.

Javascript files are the same exact thing:

```markup
<link href="/static/blog.css" rel="stylesheet">
{% if auth() %}
    <form action="/blog/create" method="POST">
        {{ csrf_field|safe }}

        <label> Title </label>
        <input type="name" name="title"><br>
        
        <label> Body </label>
        <textarea name="body"></textarea>
        
        <input type="submit" value="Post!">
    </form>
{% else %}
    <a href="/login">Please Login</a>
{% endif %}

<script src="/static/script.js"></script>
```

{% hint style="success" %}
For more information on static files, checkout the [Static Files](../the-basics/static-files.md) documentaton.
{% endhint %}

## The Controller For Creating And The Container

Notice that our action is going to `/blog/create` so we need to direct a route to our controller method. In this case we will direct it to a `store` method.

Let's open back up routes/web.py and create a new route. Just add this to the `ROUTES` list:

```python
Post().route('/blog/create', 'BlogController@store'),
```

and create a new store method on our controller:

```python
....
def show(self): 
    return view('blog')

# New store Method
def store(self): 
     pass
```

Now notice above in the form we are going to be receiving 2 form inputs: title and body. So let's import the `Post` model and create a new post with the input.

```python
from app.Post import Post
...

def store(self, Request):
    Post.create(
        title=Request.input('title'),
        body=Request.input('body'),
        author_id=Request.user().id
    )

    return 'post created'
```

Notice that we used `Request` here. This is the `Request` object. Where did this come from? This is the power and beauty of Masonite and your first introduction to the [Service Container](../architectural-concepts/service-container.md). The Service Container is an extremely powerful implementation as allows you to ask Masonite for an object \(in this case `Request`\) and get the `Request` object.

{% hint style="success" %}
Read more about the [Service Container](../architectural-concepts/service-container.md) here.
{% endhint %}

More likely, you will use the request helper and it will look something like this instead:

```python
from app.Post import Post
...

def store(self):
    Post.create(
        title=request().input('title'),
        body=request().input('body'),
        author_id=request().user().id
    )

    return 'post created'
```

We can use the `request()` function. This is what Masonite calls [Helper Functions](../the-basics/helper-functions.md) which speed up development. We didn't import anything but we are able to use them. This is because Masonite ships with a [Service Provider](../architectural-concepts/service-providers.md) that adds builtin functions to the project.

Also notice we used an input\(\) method. Masonite does not discriminate against different request methods so getting input on a GET or a POST request doesn't matter. You will always use this input method.

Go ahead and run the server using craft serve and head over to `localhost:8000/blog` and create a post. This should hit the `/blog/create` route with the `POST` request method and we should see "post created".







