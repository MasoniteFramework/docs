# Part 8 - Updating and Deleting Posts

## Getting Started

By now, all of the logic we have gone over so far will take you a long way so let's just finish up quickly with updating and deleting a posts.

## Update Controller Method

Let's just make an update method on the PostController:

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

Since we are more comfortable with controllers we can go ahead and make two at once. We made one that shows a view that updates and then one that actually updates the post.

## Create The View

```text
$ craft view update
```

```markup
<form action="/post/update" action="POST">
    {{ csrf_field|safe }}

    <label for="">Title</label>
    <input type="text" name="title"><br>

    <label>Body</label>
    <textarea name="body"></textarea><br>

    <input type="submit" value="Update">
</form>
```

## Create The Route:

Remember we made 2 controller methods so let's attach them to a route here:

```python
Get().route('/post/@id/update', 'PostController@update'),
Post().route('/post/@id/update', 'PostController@store'),
```

