# Part 7 - Showing Our Posts

## Getting Started

Lets go ahead and show how we can show the posts we just created. In this part we will create 2 new templates to show all posts and a specific post.

## Creating The Templates

Let's create 2 new templates.

```text
$ craft view posts
$ craft view single
```

Let's start with showing all posts

## Creating The Controller

Let's create a controller for the posts to separate it out from the BlogController.

```text
$ craft controller Post
```

Great! So now in our show method we will create show all posts and then we will create a single method to show a specific post.

## Show Method

Let's get the show method to return the posts view with all the posts:

```python
from app.Post import Post

...

def show(self):
    posts = Post.all()
    
    return view('posts', {'posts': posts})
```

## Posts Route

We need to add a route for this method:

```python
Get().route('/posts', 'PostController@show')
```

## Posts View

Our posts view can be very simple:

```markup
{% for post in posts %}
    {{ post.title }}
    <br>
    {{ post.body }}
    <hr>
{% endfor %}
```

Go ahead and run the server and head over the localhost:8000/posts route. You should see a basic representation of your posts. If you only see 1, go to localhost:8000/blog to create more so we can show an individual post.

Let's repeat the process but change our workflow a bit.

## Single Post Route

We need to add a route for this method:

```python
Get().route('/post/@id', 'PostController@single')
```

Notice here we have a @id string. We can use this to grab that section of the URL in our controller in the next section below.

## Single Method

Let's create a single method so we show a single post

```python
from app.Post import Post

...

def single(self):
    post = Post.find(request().param('id'))
    
    return view('single', {'post': post})
```

We use the param method to fetch the id from the URL. Remember this key was set in the route above when we specified the `@id`

{% hint style="info" %}
For a real application we might even do something like `@slug` and then fetch it with `request().param('slug')`.
{% endhint %}

## Single Post View

We just need to display 1 post so lets just put together a simple view:

```markup
{{ post.title }}
<br>
{{ post.body }}
<hr>
```

Go ahead and run the server and head over the localhost:8000/post/1 route and then localhost:8000/post/2 and see how the posts are different.



