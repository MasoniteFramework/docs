# Models

Models are the main starting point for using the ORM portion of Masonite ORM. Models are pretty simple classes to start out with. They can get more robust over time but nearly all the overhead is abstracted away for you and they allow you to override anything you need.

# Creating a Model

Models are pretty simple to create. They are simple classes that expand as you need them to. To create a model simply create a new class and extend Masonite ORM's `Model` class:

```python
from masonite.orm.models import Model

class User(Model):
    pass
```

Once created that is it. There are a few conventions you might need to take into account to get started though.

## Table Names

The `User` model we created above will use the plural version of the models name. So a `User` model will use a `users` table and a `Company` model will use a `companies` table. Masonite ORM will also use a snake case version of the model name for the table. So it will take a class name like `UserInvoice` and automatically set the table to `user_invoices`.

If you need to override this behavior you can do so by specifying the `__table__` attribute:

```python
from masonite.orm.models import Model

class User(Model):
    __table__ = 'accounts'
```

## Primary Keys

Masonite ORM will also default that all primary keys are `id`. If this is not the case you can specify the primary key column explicitly:

```python
from masonite.orm.models import Model

class User(Model):
    __table__ = 'accounts'
    __primary_key__ = 'user_id'
```

## Connections

By default, Masonite will also assume that all models are using the `default` connection which was defined in your database configuration dictionary. It is possible to have different models use difference connections by specifying the `__connection__` attribute:

```python
from masonite.orm.models import Model

class User(Model):
    __table__ = 'accounts'
    __primary_key__ = 'user_id'
    __connection__ = 'db-server-1'
```

## Retrieving Records

### Getting All Records

Once your model is created you can start querying your database. It's pretty simple to get all records:

```python
from models import User

for user in User.all():
    user.name
```

### Selecting Columns

### Subqueries

### Subgroups

### Aggregates

## Timestamps

## Inserting

### Mass Assignment

## Updating

## Deleting

# Relationships

## Making a Relationship

### Belongs To

### Has Many

## Accessing a Relationship

## Calling a Relationship

## Checking Existence

* has queries
* where_has queries
* doesnt_have queries

## Eager Loading

* with_

# Scopes

## Passing Parameters

# Global Scopes

# Global scopes

# Accessors and Mutators

# Casting

# Dates

## Setting Timezones

# Touching

# Available Methods

# Serializing and JSON

## Hiding Attributes

## Hiding Relationships

## Converting Attributes

## Adding Attributes

* set_appends

## To Dictionary

## To JSON

## Serializing Relationships

## Serializing Dates
