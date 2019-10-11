# Part 8 - Updating and Deleting Posts

## Getting Started

By now, all of the logic we have gone over so far will take you a long way so let's just finish up quickly with updating and deleting a posts. We'll assume you are comfortable with what we have learned so far so we will run through this faster since this is just more of what were in the previous parts.

## Update Controller Method

Let's just make an update method on the `PostController`:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/PostController.py" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

Since we are more comfortable with controllers we can go ahead and make two at once. We made one that shows a view that shows a form to update a post and then one that actually updates the post with the database.

## Create The View

```text
$ craft view update
```

{% code-tabs %}
{% code-tabs-item title="resources/templates/update.html" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

## Create The Routes:

Remember we made 2 controller methods so let's attach them to a route here:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/post/@id/update', 'PostController@update'),
Post().route('/post/@id/update', 'PostController@store'),
```
{% endcode-tabs-item %}
{% endcode-tabs %}

That should be it! We can now update our posts.

## Delete Method

Let's expand a bit and make a delete method.

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/PostController.py" %}
```python
def delete(self):
    post = Post.find(request().param('id'))

    post.delete()

    return 'post deleted'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Make the route:

{% code-tabs %}
{% code-tabs-item title="routes/web.py" %}
```python
Get().route('/post/@id/delete', 'PostController@delete'),
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Notice we used a `GET` route here, It would be much better to use a `POST` method but for simplicity sake will assume you can create one by now. We will just add a link to our update method which will delete the post.

## Update the Template

We can throw a delete link right inside our update template:

{% code-tabs %}
{% code-tabs-item title="resources/templates/update.html" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

Great! You now have a blog that you can use to create, view, update and delete posts! Go on to create amazing things!

