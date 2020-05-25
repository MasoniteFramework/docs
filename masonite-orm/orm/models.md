Models are the main starting point for using the ORM portion of Masonite ORM. Models are pretty simple classes to start out with. They can get more robust over time but nearly all the overhead is abstracted away for you and they allow you to override anything you need.

# Creating a Model

Models are pretty simple to create. They are simple classes that expand as you need them to. To create a model simply create a new class and extend Masonite ORM's `Model` class:

```python
from masonite.orm.models import Model

class User(Model):
    pass
```

Once created that is it. There are a few conventions you might need to take into account to get started though.

# Table Names

The `User` model we created above will use the plural version of the models name. So a `User` model will use a `users` table and a `Company` model will use a `companies` table. Masonite ORM will also use a snake case version of the model name for the table. So it will take a class name like `UserInvoice` and automatically set the table to `user_invoices`.

If you need to override this behavior you can do so by specifying the `__table__` attribute:

```python
from masonite.orm.models import Model

class User(Model):
    __table__ = 'accounts'
```

# Primary Keys

Masonite ORM will also default that all primary keys are `id`. If this is not the case you can specify the primary key column explicitly:

```python
from masonite.orm.models import Model

class User(Model):
    __table__ = 'accounts'
    __primary_key__ = 'user_id'
```

# Connections

By default, Masonite will also assume that all models are using the `default` connection which was defined in your database configuration dictionary. It is possible to have different models use difference connections by specifying the `__connection__` attribute:

```python
from masonite.orm.models import Model

class User(Model):
    __table__ = 'accounts'
    __primary_key__ = 'user_id'
    __connection__ = 'db-server-1'
```

# Retrieving Records

## Getting All Records

Once your model is created you can start querying your database. It's pretty simple to get all records:

```python
from models import User

users = User.all()

for user in users:
    user.name
```

Models are simple proxies around the (QueryBuilder)[masonite-orm/query-builder.md] class so if you need to reference any models you can simply call any query builder methods. Because of this, you can chain as many query builder methods onto the model as you want and all will be proxied to the underlying query builder:

```python
from models import User

users = (User
    .where('active', 1)
    .order_by('created_at', 'desc')
    .limit(10)
    .get())

for user in users:
    user.name
```

For additional methods for use with building queries, refer to the (QueryBuilder)[masonite-orm/query-builder.md] documentation.

# Timestamps

# Inserting

# Mass Assignment

Masonite protects against mass assignment which simply means that you cannot mass update or mass create columns that are not inside the `__fillable__` array. This is a security feature and can be explained pretty simply.

Let's say we have a `users` table like this:

```
users:

username
email
active
is_admin
```

For the most part we want to create some kind of form that allows the creation of this model (like a registration form). In our controller or wherever we are saving this new record we may have somthing like this:

```python
from models import User

def store(self, request: Request):
    user = User.create(request.all())
```

This would present a pretty big security vulnerability so a user could manipulate the form to add a `is_admin` input to the form before they submit it. So Masonite protects against this behavior and instead will simply ignore anything not explicitly in the `__fillable__` array.

This only is for mass creating and mass updating. Setting attributes on the model itself is not affected:

```python
user = User.find(1)
user.is_admin = 1
user.save()
```

# Updating

Its also simple to do an update to a model. Updating is a fairly simple process. You can either update all records:

```python
admin = {'is_admin': 1}

User.update(admin)
#== UPDATE `users` SET `users`.`is_admin` = 1
```

Or specify a where condition to update records:

```python
admin = {'is_admin': 1}

User.where('email', 'joe@masoniteproject.com').update(admin)
#== UPDATE `users` SET `users`.`is_admin` = 1 WHERE `users`.`email`
```

# Deleting

Deleting is very similiar to updating. You can specify where conditions to delete:

```python
User.where('email', 'joe@masoniteproject.com').delete()
```

# Relationships

Relations are a way to represent table relationships through Python code.

# Making a Relationship

You can easily make a relationship using various relationship scopes. 

## One To One

You can make one-to-one relationship between models using the "belongs to" relationship. Let's say we have a one-to-one relationship between users and phones. Your `User` model will look something like this:

```python
from masonite.orms.relationships import belongs_to

class User:

    @belongs_to('id', 'user_id'):
    def phone:
        from models import Phone
        return Phone
```

**It's a good idea to lazy import (import your models inside the methods) to avoid circuluar dependency issues when using models as seen above.**

Unlike Orator and Eloquent, the first and second key constraints in the decorator is ALWAYS `local_key, foreign_key`. This is to avoid confusion when specifying different relationships.

To make the keys work, the first parameter is the local related key on the current model. The second parameter is the foreign related key on the related model. So on our example, `id` is the related key on the user table (the primary key in this case) and the `user_id` is the related column on the `phones` table.

Now when you need to get a phone record for a user you can simply fetch it as an attribute on the user:

```python
user = User.find(1)

user.phone #== <models.Phone object>
user.phone.number #== 123-456-7890
```

## One To Many

Sometimes your user will have a related table where there are many records in it. This table can be something like an `articles` table where a user can have many articles.

To perform this relationship on a model we can use the "has many" relationship:

```python
from masonite.orms.relationships import belongs_to

class User:

    @has_many('id', 'user_id'):
    def articles:
        from models import Article
        return Article
```

Now you can loop through all of a users articles by calling the `articles` attribute on a user model:

```python
user = User.find(1)

for article in user.articles:
    article.title
    article.description
```

# Calling a Relationship

Sometimes you need to access a relationship but also add some additional clauses. For example you might want to get all articles that are published. You can do this by **calling** the relationship, adding any clauses you need and then calling the `get` method:

```python
user = User.find(1)

for article in user.articles().where('is_published', 1).get():
    article.title
    article.description
```

This is useful for on the fly adjustments to your code. 

# Adding Clauses Relationships

In addition to calling relationships on the fly you can also add clauses directly to the relationship definition:

```python
from masonite.orms.relationships import belongs_to

class User:

    @has_many('id', 'user_id'):
    def articles:
        from models import Article
        return Article.where('is_published', 1)
```

Now when you access this relationship using `user.articles`, the where clause will be included.

# Checking Existence

Sometimes you want to get all records where a relationship exists. For example, you might want to get all users that have articles in the first place:

## Has

```python
users = User.has('articles').get()

for user in users:
    user.name
```

## Where Has

This will only fetch users that have articles. You can also run an additional query while checking for existence:

```python
users = User.where_has('articles', lambda query: query.where('is_published', 1)).get()

for user in users:
    user.name
```

This will get all users that have published articles.

# Eager Loading

Eager loading is the act of fetching various values before looping over the query. For example, let's take the example of fetching the phone for all users again (a one-to-one relationship):

```python
users = User.all() # == 5 users

for user in users:
   print(user.phone.number)
```

This will result in the following queries:

```sql
SELECT * FROM `users`
SELECT * `phones` where `phones`.`user_id` = `1`
SELECT * `phones` where `phones`.`user_id` = `2`
SELECT * `phones` where `phones`.`user_id` = `3`
SELECT * `phones` where `phones`.`user_id` = `4`
SELECT * `phones` where `phones`.`user_id` = `5`
```

Notice it is doing 1 query for each user we are iterating over. This is known as the "N + 1" problem. The "N" is the number of users we are iterating over and the "1" is the first query we are doing to make the first query.

To solve this, we can do something called "eager loading" which simply means we fetch all the phones for our users first and then we can loop over that in memory rather than making costly database calls. This results in a faster application and fewer database calls:

```python
users = User.with_('phone').all() # == 5 users

for user in users:
   print(user.phone.number)
```

This will now result in the following queries:

```sql
SELECT * FROM `users`
SELECT * `phones` where `phones`.`user_id` IN ('1', '2', '3', '4', '5')
```

Notice how we take the result of the second query and load the result into the model. Then when we fetch the `phone` attribute, the result is found in memory rather than making subsquent calls to the database.

# Scopes

Scopes are an excellent way to abstract out recurring statesments and make them much more readable. For example, we have been specifying a `is_active` column in our where clauses for our users. We can turn this query here:

```
users = User.where('is_active', 1).get()
```

Into this:

```
users = User.is_active().get()
```

and put all the where logic into a query scope. To do this we can simply create the scope on our `User` model:

```python
from masonite.orm.scopes import scope

class User(Model):

    @scope
    def is_active(self, query):
        return query.where('is_active', 1)
```

## Passing Parameters

You can also pass variables into your scopes if you need to keep them abstract:

```python
from masonite.orm.scopes import scope

class User(Model):

    @scope
    def gender(self, query, gender):
        return query.where('gender', gender)
```

And can pass in parameters as normal:

```python
users = User.gender('M').get()
users = User.gender('F').get()
```

# Global Scopes

# Accessors

Accessors are called in place of the attribute you want to access. For example, if you want to access `first_name` but you want to add some additional logic to it you can simply create an accessor method for it. To do this you just need to specify a method that starts with `get_`

```python
class User(Model):

    def get_first_name(self):
        return "Hello, " + self.get_raw_attribute('first_name')
```

You can see if we don't specify `get_raw_attribute` then we will end up in an infinite loop

# Casting Attributes

Some values you may need to be casted to other values. For example, you may have an `is_admin` column that is either a `1` or `0`. It might be more useful to cast this to a `True` or `False` boolean value when you access it. You can create specific castings:

```python
class User(Model):
    __casts__ = {"is_admin": "bool"}
```

The cast methods (the dictionary values here) are predefined. The available casting methods are:

* bool
* json

## Creating Casting Methods

You can create your own casting methods as well. If you want to convert a string value to an integer value for example you can create your own casting class:

```python
class IntCast:

    def get(self, value):
        return int(value)
```

and then register it to your model:

```python
from casts import IntCast

class User(Model):
    __cast_map__ = {
        'int': IntCast
    }

    __casts__ = {"age": "int"}
```

# Timestamps

By default, Masonite assumes that there are `created_at` and `updated_at` columns and will manage those for you automatically. If you don't want Masonite to manage those columns or your table does not have those columns, you can turn this off:

```python
from masonite.orm.scopes import scope

class User(Model):
    __timestamps__ = None
```

If your columns are not called `created_at` or `updated_at` you can change the name of the columns:

```python
from masonite.orm.scopes import scope

class User(Model):
    date_created_at = "created_at"
    date_updated_at = "updated_at"
```

# Dates

Masonite needs to know explicitly which columns are date columns. You can do so directly on the model:

```python
class Article(Model):

    __dates__ = ['published_at', 'printed_at']
```

This will handle both setting and accessing dates on your models

# Setting Timezones

In development. Not currently possible.

# Touching

If you just want to update the `updated_at` column quickly you can use the `touch` method:

```python
user = User.find(1).touch()
```

This will update the `updated_at` column on your table to the current time.


# Serializing and JSON

If you need to convert your models (and their relationships) to a dictionary you can do so simple:

```python
user = User.find(1)
user.serialize()
```

This will convert the model into a dictionary. You can also do the same thing with a collection:

```python
users = User.all()
users.serialize()
```

This will convert into a list of dictionaries.

## Serializing Relationships

You can automatically serialize relationships as well but specifying them in the with:

```python
users = User.with_('articles').get()
users.serialize()
```

This list of dictionaries will now also have the correct relationship results attached to each value

## Hiding Attributes And Relationships

Sometimes you don't want to return some results from the dictionary. For example, you may be building an API and don't want to show the user's password.

You can do so by specifying the `__hidden__` attribute before serializing:

```python
user = User.find(1)
user.__hidden__ = ['password']
user.serialize()
```

This also works with relationships as well.

# Adding Attributes

Sometimes you will want to add extra attributes to a serialization. For example, maybe you want to add the fact that a user is subscribed:

First you will need to set a `@property` attribute on your model:

```python
class User(Model):

    @property
    def is_subscribed(self):
        return {
            'subscribed': True
        }
```

then you can use the `set_appends` method before serializing:

```python
user = User.find(1)
user.set_appends(['is_subscribed'])
user.serialize()
```

This will now attach the `is_subscribed` response to the dictionary.

# To JSON

Not currently available

# Serializing Dates

If you need to serialize dates to a specific format you can override a few methods. Whenever Masonite serializes a model, for all the dates in the `__dates__` list it will pass those values into a `get_new_serialized_date` method.

You can override this method and change the format of the dates:

```python
class Article(Model):

    __dates__ = ['completed_at', 'published_at']

    def get_new_serialized_date(self, datetime):
        datetime #== instance of datetime.datetime from python library
        return pendulum.instance(datetime).to_datetime_string()
```
