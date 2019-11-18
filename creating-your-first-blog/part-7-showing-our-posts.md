# Part 7 - Showing Our Posts

## Getting Started

Lets go ahead and show how we can show the posts we just created. In this part we will create 2 new templates to show all posts and a specific post.

## Creating The Templates

Let's create 2 new templates.

{% code title="terminal" %}
```text
$ craft view posts
$ craft view single
```
{% endcode %}

Let's start with showing all posts

## Creating The Controller

Let's create a controller for the posts to separate it out from the `BlogController`.

{% code title="terminal" %}
```text
$ craft controller Post
```
{% endcode %}

Great! So now in our `show` method we will show all posts and then we will create a `single` method to show a specific post.

## Show Method

Let's get the `show` method to return the posts view with all the posts:

{% code title="app/http/controllers/PostController.py" %}
```python
from app.Post import Post

...

def show(self):
    posts = Post.all()
    
    return view('posts', {'posts': posts})
```
{% endcode %}

## Posts Route

We need to add a route for this method:

{% code title="routes/web.py" %}
```python
Get().route('/posts', 'PostController@show')
```
{% endcode %}

## Posts View

Our posts view can be very simple:

{% code title="resources/templates/posts.html" %}
```markup
{% for post in posts %}
    {{ post.title }}
    <br>
    {{ post.body }}
    <hr>
{% endfor %}
```
{% endcode %}

Go ahead and run the server and head over to `localhost:8000/posts` route. You should see a basic representation of your posts. If you only see 1, go to `localhost:8000/blog` to create more so we can show an individual post.

### Showing The Author

Remember we made our author relationship before. Orator will take that relationship and make an attribute from it so we can display the author's name as well:

{% code title="resources/templates/posts.html" %}
```markup
{% for post in posts %}
    {{ post.title }} by {{ post.author.name }}
    <br>
    {{ post.body }}
    <hr>
{% endfor %}
```
{% endcode %}

Let's repeat the process but change our workflow a bit.

## Single Post Route

Next we want to just show a single post. We need to add a route for this method:

{% code title="routes/web.py" %}
```python
Get().route('/post/@id', 'PostController@single')
```
{% endcode %}

Notice here we have a `@id` string. We can use this to grab that section of the URL in our controller in the next section below.

## Single Method

Let's create a `single` method so we show a single post.

{% code title="app/http/controllers/PostController.py" %}
```python
from app.Post import Post

...

def single(self):
    post = Post.find(request().param('id'))
    
    return view('single', {'post': post})
```
{% endcode %}

We use the `param` method to fetch the id from the URL. Remember this key was set in the route above when we specified the `@id`

{% hint style="info" %}
For a real application we might do something like `@slug` and then fetch it with `request().param('slug')`.
{% endhint %}

## Single Post View

We just need to display 1 post so lets just put together a simple view:

{% code title="resources/templates/single.html" %}
```markup
{{ post.title }}
<br>
{{ post.body }}
<hr>
```
{% endcode %}

Go ahead and run the server and head over the `localhost:8000/post/1` route and then `localhost:8000/post/2` and see how the posts are different.



