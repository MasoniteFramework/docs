# Orator To Masonite ORM Guide

This guide will explain how to move from Orator to Masonite ORM. Masonite ORM was made to be pretty much a straight port of Orator but allow the Masonite organization complete creative control of the ORM.

Before moving your project over to Masonite ORM please keep in mind some features are not _\(_at least currently_\)_ ported over from Orator. These are features that may be ported over in the future. Currently these features are:

* through relationships (HasOneThrough, HasManyThrough, etc)

## Config

The configuration dictionary between Orator and Masonite ORM is identical. The only difference is that Masonite ORM requires a `config/database.py` file whereas Orator was optional and needed to be explicitly specified in several places like commands.

If you are coming from Masonite already then don't worry, this file is already there. If not you will need to create this `config/database.py` file.

This is an example of a Masonite ORM config dictionary:


```python
import os

DATABASES = {
    'default': 'mysql',
    'mysql': {
        'driver': 'mysql',
        'host': os.getenv('MYSQL_DATABASE_HOST'),
        'user': os.getenv('MYSQL_DATABASE_USER'),
        'password': os.getenv('MYSQL_DATABASE_PASSWORD'),
        'database': os.getenv('MYSQL_DATABASE_DATABASE'),
        'port': os.getenv('MYSQL_DATABASE_PORT'),
        'prefix': '',
        'options': {
            'charset': 'utf8mb4',
        },
        'log_queries': True
    },
    'postgres': {
        'driver': 'postgres',
        'host': os.getenv('POSTGRES_DATABASE_HOST'),
        'user': os.getenv('POSTGRES_DATABASE_USER'),
        'password': os.getenv('POSTGRES_DATABASE_PASSWORD'),
        'database': os.getenv('POSTGRES_DATABASE_DATABASE'),
        'port': os.getenv('POSTGRES_DATABASE_PORT'),
        'prefix': '',
        'log_queries': True
    },
    'sqlite': {
        'driver': 'sqlite',
        'database': 'orm.sqlite3',
        'prefix': '',
        'log_queries': True
    },
    'mssql': {
        'driver': 'mssql',
        'host': os.getenv('MSSQL_DATABASE_HOST'),
        'user': os.getenv('MSSQL_DATABASE_USER'),
        'password': os.getenv('MSSQL_DATABASE_PASSWORD'),
        'database': os.getenv('MSSQL_DATABASE_DATABASE'),
        'port': os.getenv('MSSQL_DATABASE_PORT'),
        'prefix': '',
        'log_queries': True
    },
}
```

The other thing you will need to do is change the resolver classes. Orator has a configuration structure like this:

```python
from orator import DatabaseManager, Model

DATABASES = {
  # ...
}

DB = DatabaseManager(DATABASES)
Model.set_connection_resolver(DB)
```

Masonite ORM those same resolver classes looks like this:

```python
from masoniteorm.connections import ConnectionResolver

DATABASES = {
  # ...
}

db = ConnectionResolver().set_connection_details(DATABASES)
```

## Models

Models are identical but the imports are different. Orator requires you to set the model resolver from the configuration file and then you import that model.

In Masonite ORM you import the model directly:

```python
# Masonite
from masoniteorm.models import Model

class User(Model):
    pass
```

## Scopes

Scopes are also identical but the import changes:

```python
# Orator
from orator.orm import scope

class User(Model):

  @scope
  def popular(self, query):
        return query.where('votes', '>', 100)
```

```python
# Masonite
from masoniteorm.scopes import scope

class User(Model):

  @scope
  def popular(self, query):
        return query.where('votes', '>', 100)
```

## Relationships

Relationships are also slightly different. In Orator there is a `has_one` relationship and a `belongs_to` relationship. In Masonite ORM this is only a belongs_to relationship. The logic behind has_one and belongs_to is generally identical so there was no reason to port over has_one other than for semantical purposes.

## BelongsTo and HasOne

So if you have something like this in your Orator codebase:

```python
# Orator
from orator.relationships import has_one

class User(Model):

  @has_one('other_key', 'local_key')
  def phone(self):
      from app.models import Phone
      return Phone
```

It will now become:

```python
# Orator
from masoniteorm.relationships import belongs_to

class User(Model):

  @belongs_to('local_key', 'other_key') # notice the keys also switched places
  def phone(self):
      from app.models import Phone
      return Phone
```

## Relationship Keys

In Orator, some relationships require a specific order of keys. For example a belongs to relationship is `belongs_to('local_key', 'other_key')`but a has one is `has_one('other_key', 'local_key')`. This is very confusing to remember so in Masonite ORM the keys are always `local_key, other_key`. 

# Fetching builder relations

In Orator you could do this:

```python
user = User.find(1)
user.phone().where('active', 1).get()
```

This would delay the relationship call and would instead append the builder before returning the result. 

The above call in Masonite ORM becomes:

```python
user = User.find(1)
user.related('phone').where('active', 1).get()
```

