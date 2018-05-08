# Part 5 - Models


## Getting Started

Now that we have our tables and migrations all done and we have a posts table, let's create a model for it.

Models in Masonite are a bit different than other Python frameworks. Masonite uses Orator which is an Active Record implementation of an ORM. This bascially means we will not be building our model and then translating that into a migration. Models and migrations are separate in Masonite. Our models will take shape of our tables regardless of what the table looks like.

## Creating our Model

Again, we can use a craft command to create our model:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft model Post
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Notice we used the singular form for our model. By default, Orator will check for the plural name of the class in our database \(in this case posts\) by simply appending an "s" onto the model. We will talk about how to specify the table explicitly in a bit.

The model created now resides inside `app/Post.py` and when we open it up it should look like:

{% code-tabs %}
{% code-tabs-item title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Simple enough, right? Like previously stated, we don't have to manipulate the model. The model will take shape of the table as we create or change migrations.

## Table Name

Again, the table name that the model is attached to is the plural version of the model \(by appending an "s"\) but if you called your table something different such as "blog" instead of "blogs" we can specify the table name explicitly:

{% code-tabs %}
{% code-tabs-item title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    __table__ = 'blog'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Mass Assignment

Orator by default protects against mass assignment as a security measure so we will explicitly need to set what columns we would like to be fillable:

{% code-tabs %}
{% code-tabs-item title="app/Post.py" %}
```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    __fillable__ = ['title', 'author_id', 'body']
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Relationships

The relationship is pretty straight forward here. Remember that we created a foreign key in our migration. We can create that relationship in our model like so:

{% code-tabs %}
{% code-tabs-item title="app/Post.py" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

Because of how Masonite does models, it is typically better to perform the import inside the relationship like we did above. If you import at the top you may encounter circular imports. 

{% hint style="success" %}
We won't go into much more detail here about different types of relationships but to learn more, read the [ORM](https://orator-orm.com/docs/0.9/orm.html) documentation.
{% endhint %}


