# Creating a Blog

## Preface

This section of the documentation will contain various tutorials. These are guides that are designed to take you from beginning to end on building various types of projects with Masonite. We may not explain things in much detail for each section as this part of the documentation is designed to just get you familiar with the inner workings of Masonite.

Since this section of the documentation is designed to just get you up and coding with Masonite, any further explanations that should be presented are inside various "hint blocks." Once you are done with the tutorial or simply want to learn more about a topic it is advised that you go through each hint block and follow the links to dive deeper into the reference documentation which does significantly more explaining.

## Hint Blocks

You will see various hint blocks throughout the tutorials. Below are examples of what the various colors represent.

{% hint style="success" %}
You'll see hint blocks that are green which you should follow if you want to learn more information about the topic currently being discussed.
{% endhint %}

{% hint style="info" %}
You'll also see hint blocks that are blue. These should not be ignored and typically contain background information you need to further understand something.
{% endhint %}

## Installation

This tutorial will assume you have already installed Masonite. If you haven't, be sure to read the [Installation](../) guide to get a fresh install of Masonite up and running. Once you have one up and running or if you already have it running, go ahead and continue on.

## Creating a Blog

In this tutorial we will walk through how to create a blog. We will touch on all the major systems of Masonite and it should give you the confidence to try the more advanced tutorials or build an application yourself.

## Routing

Typically your first starting point for your Masonite development flow will be to create a route. All routes are located in `routes/web.py` and are extremely simple to understand. They consist of a request method and a route method. Routing is simply stating what incoming URI's should direct to which controllers.

For example, to create a `GET` request route it will look like:

{% code title="routes/web.py" %}
```python
Get().route('/url', 'Controller@method')
```
{% endcode %}

We'll talk more about the controller in a little bit.

{% hint style="success" %}
You can read more about routes in the [Routing](../the-basics/routing.md) documentation
{% endhint %}

### Creating our Route:

We will start off by creating a view and controller to create a blog post.

A controller is a simple class that holds controller methods. These controller methods will be what our routes will call so they will contain all of our application's business logic.

{% hint style="info" %}
Think of a controller method as a function in the `views.py` file if you are coming from the Django framework
{% endhint %}

Let's create our first route now. We can put all routes inside `routes/web.py` and inside the `ROUTES` list. You'll see we have a route for the home page. Let's add a route for creating blogs.

{% code title="routes/web.py" %}
```python
ROUTES = [
    Get().route('/', 'WelcomeController@show').name('welcome'),

    # Blog Routes
    Get().route('/blog', 'BlogController@show')
]
```
{% endcode %}

You'll notice here we have a `BlogController@show` string. This means "use the blog controller's show method to render this route". The only problem here is that we don't yet have a blog controller.

{% hint style="success" %}
Let's create the `BlogController` in the next step: [Part 2 - Creating Our First Controller](creating-a-blog.md)
{% endhint %}

## Creating a Controller

All controllers are located in the `app/http/controllers` directory and Masonite promotes 1 controller per file. This has proven efficient for larger application development because most developers use text editors with advanced search features such as Sublime, VSCode or Atom. Switching between classes in this instance is simple and promotes faster development. It's easy to remember where the controller exactly is because the name of the file is the controller.

You can of course move controllers around wherever you like them but the craft command line tool will default to putting them in separate files. If this seems weird to you it might be worth a try to see if you like this opinionated layout.

### Creating Our First Controller

Like most parts of Masonite, you can scaffold a controller with a craft command:

{% code title="terminal" %}
```text
$ craft controller Blog
```
{% endcode %}

This will create a controller in `app/http/controllers` directory that looks like:

{% code title="app/http/controller/BlogController.py" %}
```python
class BlogController:
    ''' Class Docstring Description '''

    def show(self):
        pass
```
{% endcode %}

Simple enough, right? You'll notice we have a `show` method. These are called "controller methods" and are similiar to what Django calls a "view."

Notice we now have our show method that we specified in our route earlier.

### Returning a View

We can return a view from our controller. A view in Masonite are html files or "templates". They are not Python objects themselves like other Python frameworks. Views are what the users will see.

This is important as this is our first introduction to Python's IOC container. We specify in our parameter list that we need a view class and Masonite will inject it for us:

{% code title="app/http/controllers/BlogController.py" %}
```python
from masonite.view import View 

def show(self, view: View):
    return view.render('blog')
```
{% endcode %}

Notice here we annotated the `View` class. This is what Masonite call's "Auto resolving dependency injection". If you don't like the semantics of this there are other ways to "resolve" from the container that you will discover in the reference documentation but for now let's stay with this method of resolving.

{% hint style="success" %}
Be sure to learn more about the [Service Container](https://docs.masoniteproject.com/architectural-concepts/service-container).
{% endhint %}

### Creating Our View

You'll notice now that we are returning the `blog` view but it does not exist yet.

All views are in the `resources/templates` directory. We can create a new file called `resources/templates/blog.html` or we can use another craft command:

{% code title="terminal" %}
```text
$ craft view blog
```
{% endcode %}

This will create that template we wanted above for us.

We can put some text in this file like:

{% code title="resources/templates/blog.html" %}
```markup
This is a blog
```
{% endcode %}

and then run the server

{% code title="terminal" %}
```text
$ craft serve
```
{% endcode %}

and open up `localhost:8000/blog`, we will see "This is a blog"

## Authentication

Most applications will require some form of authentication. Masonite comes with a craft command to scaffold out an authentication system for you. This should typically be ran on fresh installations of Masonite since it will create controllers routes and views for you.

For our blog, we will need to setup some form of registration so we can get new users to start posting to our blog. We can create an authentication system by running the craft command:

{% code title="terminal" %}
```text
$ craft auth
```
{% endcode %}

We should get a success message saying that some new assets were created. You can check your controllers folder and you should see a few new controllers there that should handle registrations.

We will observe what was created for us in a bit.

### Database Setup

In order to register these users, we will need a database. Hopefully you already have some kind of local database setup like MySQL or Postgres but we will assume that you do not. In this case we can just use SQLite.

Now we just need to change a few environment variables so Masonite can create the SQLite database. These environment variable can be found in the `.env` file in the root of the project. Open that file up and you should see a few lines that look like:

{% code title=".env" %}
```text
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=masonite
DB_USERNAME=root
DB_PASSWORD=root
```
{% endcode %}

Go ahead and change those setting to your connection settings by adding `sqlite` to the `DB_CONNECTION` variable and whatever you want for your database which will be created for you when you migrate. We will call it `blog.db`:

{% code title=".env" %}
```text
DB_CONNECTION=sqlite
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=blog.db
DB_USERNAME=root
DB_PASSWORD=root
```
{% endcode %}

### Migrating

Once you have set the correct credentials, we can go ahead and migrate the database. Out of the box, Masonite has a migration for a users table which will be the foundation of our user. You can edit this user migration before migrating but the default configuration will suit most needs just fine and you can always add or remove columns at a later date.

{% code title="terminal" %}
```text
$ craft migrate
```
{% endcode %}

This will create our users table for us along with a migrations table to keep track of any migrations we add later.

### Creating Users

Now that we have the authentication and the migrations all migrated in, let's create our first user. Remember that we ran `craft auth` so we have a few new templates and controllers.

Go ahead and run the server:

{% code title="terminal" %}
```text
$ craft serve
```
{% endcode %}

and head over to [localhost:8000/register](http://localhost:8000/register) and fill out the form. You can use whatever name and email you like but for this purpose we will use:

```text
Username: demo
Email: demo@email.com
Password: password
```

## Migrations

Now that we have our authentication setup and we are comfortable with migrating our migrations, let's create a new migration where we will store our posts.

Our posts table should have a few obvious columns that we will simplify for this tutorial part. Let's walk through how we create migrations with Masonite.

### Craft Command

Not surprisingly, we have a craft command to create migrations. You can read more about [Database Migrations here](../orator-orm/database-migrations.md) but we'll simplify it down to the command and explain a little bit of what's going on:

{% code title="terminal" %}
```text
$ craft migration create_posts_table --create posts
```
{% endcode %}

This command simply creates the basis of a migration that will create the posts table. By convention, table names should be plural \(and model names should be singular but more on this later\).

This will create a migration in the `databases/migrations` folder. Let's open that up and starting on line 6 we should see something that looks like:

{% code title="databases/migrations/2018\_01\_09\_043202\_create\_posts\_table.py" %}
```python
def up(self):
    """
    Run the migrations.
    """
    with self.schema.create('posts') as table:
        table.increments('id')
        table.timestamps()
```
{% endcode %}

Lets add a title, an author, and a body to our posts tables.

{% code title="databases/migrations/2018\_01\_09\_043202\_create\_posts\_table.py" %}
```python
def up(self):
    """
    Run the migrations.
    """
    with self.schema.create('posts') as table:
        table.increments('id')
        table.string('title')

        table.integer('author_id').unsigned()
        table.foreign('author_id').references('id').on('users')

        table.string('body')
        table.timestamps()
```
{% endcode %}

{% hint style="success" %}
This should be fairly straight forward but if you want to learn more, be sure to read the [Database Migrations](../orator-orm/database-migrations.md) documentation.
{% endhint %}

Now we can migrate this migration to create the posts table

{% code title="terminal" %}
```text
$ craft migrate
```
{% endcode %}

## Models

Now that we have our tables and migrations all done and we have a posts table, let's create a model for it.

Models in Masonite are a bit different than other Python frameworks. Masonite uses Orator which is an Active Record implementation of an ORM. This bascially means we will not be building our model and then translating that into a migration. Models and migrations are separate in Masonite. Our models will take shape of our tables regardless of what the table looks like.

### Creating our Model

Again, we can use a craft command to create our model:

{% code title="terminal" %}
```text
$ craft model Post
```
{% endcode %}

Notice we used the singular form for our model. By default, Orator will check for the plural name of the class in our database \(in this case posts\) by simply appending an "s" onto the model. We will talk about how to specify the table explicitly in a bit.

The model created now resides inside `app/Post.py` and when we open it up it should look like:

{% code title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    pass
```
{% endcode %}

Simple enough, right? Like previously stated, we don't have to manipulate the model. The model will take shape of the table as we create or change migrations.

### Table Name

Again, the table name that the model is attached to is the plural version of the model \(by appending an "s"\) but if you called your table something different such as "blog" instead of "blogs" we can specify the table name explicitly:

{% code title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    __table__ = 'blog'
```
{% endcode %}

### Mass Assignment

Orator by default protects against mass assignment as a security measure so we will explicitly need to set what columns we would like to be fillable:

{% code title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    __fillable__ = ['title', 'author_id', 'body']
```
{% endcode %}

### Relationships

The relationship is pretty straight forward here. Remember that we created a foreign key in our migration. We can create that relationship in our model like so:

{% code title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model
from orator.orm import belongs_to

class Post(Model):
    __fillable__ = ['title', 'author_id', 'body']

    @belongs_to('author_id', 'id')
    def author(self):
        from app.User import User
        return User
```
{% endcode %}

Because of how Masonite does models, some models may rely on each other so it is typically better to perform the import inside the relationship like we did above to prevent any possibilities of circular imports.

{% hint style="success" %}
We won't go into much more detail here about different types of relationships but to learn more, read the [ORM](https://orator-orm.com/docs/0.9/orm.html) documentation.
{% endhint %}

## Designing Our Blog

Let's setup a little HTML so we can learn a bit more about how views work. In this part we will setup a really basic template in order to not clog up this part with too much HTML but we will learn the basics enough that you can move forward and create a really awesome blog template \(or collect one from the internet\).

Now that we have all the models and migrations setup, we have everything in the backend that we need to create a layout and start creating and updating blog posts.

We will also check if the user is logged in before creating a template.

### The Template For Creating

The template for creating will be located at `/blog/create` and will be a simple form for creating a blog post

{% code title="resources/templates/blog.html" %}
```markup
<form action="/blog/create" method="POST">
    {{ csrf_field }}

    <input type="name" name="title">
    <textarea name="body"></textarea>
</form>
```
{% endcode %}

Notice here we have this strange `{{ csrf_field }}` looking text. Masonite comes with CSRF protection so we need a token to render with the CSRF field.

Now because we have a foreign key in our posts table, we need to make sure the user is logged in before creating this so let's change up our template a bit:

{% code title="resources/templates/blog.html" %}
```markup
{% if auth() %}
    <form action="/blog/create" method="POST">
        {{ csrf_field }}

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
{% endcode %}

`auth()` is a globally available function that either returns the current user or returns `None`.

{% hint style="success" %}
Masonite uses Jinja2 templating so if you don't understand this templating, be sure to [read their documentation](http://jinja.pocoo.org/docs/2.10/).
{% endhint %}

### Static Files

For simplicity sake, we won't be styling our blog with something like Bootstrap but it is important to learn how static files such as CSS files work with Masonite so let's walk through how to add a CSS file and add it to our blog.

Firstly, head to storage/static/ and make a blog.css file and throw anything you like in it. For this tutorial we will make the html page slightly grey.

{% code title="storage/static/blog.css" %}
```css
html {
    background-color: #ddd;
}
```
{% endcode %}

Now we can add it to our template like so right at the top:

{% code title="resources/templates/blog.html" %}
```markup
<link href="/static/blog.css" rel="stylesheet">
{% if auth() %}
    <form action="/blog/create" method="POST">
        {{ csrf_field }}

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
{% endcode %}

That's it. Static files are really simple. It's important to know how they work but for this tutorial we will ignore them for now and focus on more of the backend.

Javascript files are the same exact thing:

{% code title="resources/templates/blog.html" %}
```markup
<link href="/static/blog.css" rel="stylesheet">
{% if auth() %}
    <form action="/blog/create" method="POST">
        {{ csrf_field }}

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
{% endcode %}

{% hint style="success" %}
For more information on static files, checkout the [Static Files](../the-basics/static-files.md) documentaton.
{% endhint %}

### The Controller For Creating And The Container

Notice that our action is going to `/blog/create` so we need to direct a route to our controller method. In this case we will direct it to a `store` method.

Let's open back up routes/web.py and create a new route. Just add this to the `ROUTES` list:

{% code title="routes/web.py" %}
```python
Post().route('/blog/create', 'BlogController@store'),
```
{% endcode %}

and create a new store method on our controller:

{% code title="app/http/controllers/BlogController.py" %}
```python
....
def show(self): 
    return view('blog')

# New store Method
def store(self): 
     pass
```
{% endcode %}

Now notice above in the form we are going to be receiving 2 form inputs: title and body. So let's import the `Post` model and create a new post with the input.

{% code title="app/http/controllers/BlogController.py" %}
```python
from app.Post import Post
from masonite.request import Request
...

def store(self, request: Request):
    Post.create(
        title=request.input('title'),
        body=request.input('body'),
        author_id=request.user().id
    )

    return 'post created'
```
{% endcode %}

Notice that we used `Request` here. This is the `Request` object. Where did this come from? This is the power and beauty of Masonite and your first introduction to the [Service Container](../architectural-concepts/service-container.md). The [Service Container](../architectural-concepts/service-container.md) is an extremely powerful implementation as allows you to ask Masonite for an object \(in this case `Request`\) and get that object. This is an important concept to grasp so be sure to read the documentation further.

{% hint style="success" %}
Read more about the [Service Container](../architectural-concepts/service-container.md) here.
{% endhint %}

More likely, you will use the request helper and it will look something like this instead:

{% code title="app/http/controllers/BlogController.py" %}
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
{% endcode %}

Notice we used the `request()` function. This is what Masonite calls [Helper Functions](../the-basics/helper-functions.md) which speed up development. We didn't import anything but we are able to use them. This is because Masonite ships with a [Service Provider](../architectural-concepts/service-providers.md) that adds builtin functions to the project.

Also notice we used an `input()` method. Masonite does not discriminate against different request methods so getting input on a `GET` or a `POST` request doesn't matter. You will always use this input method.

Go ahead and run the server using craft serve and head over to `localhost:8000/blog` and create a post. This should hit the `/blog/create` route with the `POST` request method and we should see "post created".

## Showing Our Posts

Lets go ahead and show how we can show the posts we just created. In this part we will create 2 new templates to show all posts and a specific post.

### Creating The Templates

Let's create 2 new templates.

{% code title="terminal" %}
```text
$ craft view posts
$ craft view single
```
{% endcode %}

Let's start with showing all posts

### Creating The Controller

Let's create a controller for the posts to separate it out from the `BlogController`.

{% code title="terminal" %}
```text
$ craft controller Post
```
{% endcode %}

Great! So now in our `show` method we will show all posts and then we will create a `single` method to show a specific post.

### Show Method

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

### Posts Route

We need to add a route for this method:

{% code title="routes/web.py" %}
```python
Get().route('/posts', 'PostController@show')
```
{% endcode %}

### Posts View

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

#### Showing The Author

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

### Single Post Route

Next we want to just show a single post. We need to add a route for this method:

{% code title="routes/web.py" %}
```python
Get().route('/post/@id', 'PostController@single')
```
{% endcode %}

Notice here we have a `@id` string. We can use this to grab that section of the URL in our controller in the next section below.

### Single Method

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

We use the `param()` method to fetch the id from the URL. Remember this key was set in the route above when we specified the `@id`

{% hint style="info" %}
For a real application we might do something like `@slug` and then fetch it with `request().param('slug')`.
{% endhint %}

### Single Post View

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

## Updating and Deleting Posts

By now, all of the logic we have gone over so far will take you a long way so let's just finish up quickly with updating and deleting a posts. We'll assume you are comfortable with what we have learned so far so we will run through this faster since this is just more of what were in the previous parts.

### Update Controller Method

Let's just make an update method on the `PostController`:

{% code title="app/http/controllers/PostController.py" %}
```python
def update(self):
    post = Post.find(request().param('id'))

    return view('update', {'post': post})

def store(self):
    post = Post.find(request().param('id'))

    post.title = request().input('title')
    post.body = request().input('body')

    post.save()

    return 'post updated'
```
{% endcode %}

Since we are more comfortable with controllers we can go ahead and make two at once. We made one that shows a view that shows a form to update a post and then one that actually updates the post with the database.

### Create The View

```text
$ craft view update
```

{% code title="resources/templates/update.html" %}
```markup
<form action="/post/{{ post.id }}/update" method="POST">
    {{ csrf_field }}

    <label for="">Title</label>
    <input type="text" name="title" value="{{ post.title }}"><br>

    <label>Body</label>
    <textarea name="body">{{ post.body }}</textarea><br>

    <input type="submit" value="Update">
</form>
```
{% endcode %}

### Create The Routes:

Remember we made 2 controller methods so let's attach them to a route here:

{% code title="routes/web.py" %}
```python
Get().route('/post/@id/update', 'PostController@update'),
Post().route('/post/@id/update', 'PostController@store'),
```
{% endcode %}

That should be it! We can now update our posts.

### Delete Method

Let's expand a bit and made a delete method.

{% code title="app/http/controllers/PostController.py" %}
```python
def delete(self):
    post = Post.find(request().param('id'))

    post.delete()

    return 'post deleted'
```
{% endcode %}

### Make the route:

{% code title="routes/web.py" %}
```python
Get().route('/post/@id/delete', 'PostController@delete'),
```
{% endcode %}

Notice we used a `GET` route here, It would be much better to use a `POST` method but for simplicity sake will assume you can create one by now. We will just add a link to our update method which will delete the post.

### Update the Template

We can throw a delete link right inside our update template:

{% code title="resources/templates/update.html" %}
```markup
<form action="/post/{{ post.id }}/update" method="POST">
    {{ csrf_field }}

    <label for="">Title</label>
    <input type="text" name="title" value="{{ post.title }}"><br>

    <label>Body</label>
    <textarea name="body">{{ post.body }}</textarea><br>

    <input type="submit" value="Update">

    <a href="/post/{{ post.id }}/delete"> Delete </a>
</form>
```
{% endcode %}

Great! You now have a blog that you can use to create, view, update and delete posts! Go on to create amazing things!

