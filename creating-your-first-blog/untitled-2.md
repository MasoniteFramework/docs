# Part 5 - Models

{% hint style="danger" %}
This section is incomplete
{% endhint %}

## Getting Started

Now that we have our tables and migrations all done and we have a posts table, let's create a model for it.

Models in Masonite are a bit different than other Python frameworks. Masonite uses Orator which is an Active Record implementation of an ORM. This bascially means we will not be building our model and then translating that into a migration. Models and migrations are separate in Masonite. Our models will take shape of our tables regardless of what the table looks like.

## Creating our Model

Again, we can use a craft command to create our model:

```text
$ craft model Post
```

Notice we used the singular form for our model. By default, Orator will check for the plural name of the class in our database \(in this case posts\) by simply appending an "s" onto the model. We will talk about how to specify the table explicitly in a bit.

The model created now resides inside app/Post.py and when we open it up it should look like:

```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    pass
```

Simple enough, right? Like previously stated, we don't have to manipulate the model. The model will take shape of the table as we create or change migrations.

## Mass Assignment

Orator by default protects against mass assignment as a security measure so we will explicitly need to set what columns we would like to be fillable:

```python
''' A Post Database Model '''
from config.database import Model

class Post(Model):
    __fillable__ = ['title', 'author_id', 'body']
```



