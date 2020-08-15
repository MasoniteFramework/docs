# Orator To Masonite ORM

## Masonite To Orator Upgrade Guide

This guide will explain how to move from Orator to Masonite ORM. Masonite ORM was made to be pretty much a straight port of Orator but allow the Masonite organization complete creative control of the ORM.

Before moving your project over to Masonite ORM please keep in mind some features are not \(at least currently\) ported over from Orator. These are features that may be ported over in the future. These features are:

* Pagination
* Inserting related models
* Pessimistic Locking
* Database Transactions
* Model Events and Observers
* `.change()` functionality in migrations
* polymorphic relationships
* through relationships

## Config

The configuration dictionary between Orator and Masonite ORM is identical. The only difference is that Masonite ORM **requires** a `config/database.py` file whereas Orator was optional and needed to be explicitly specified in several places.

If you are coming from Masonite already then don't worry, this file is already there. If not you will need to create this `config/database.py` file.

This is an example of a Masonite ORM config dictionary:

```python
# config/database.py
DATABASES = {
    'default': 'mysql',
    'mysql': {
        'driver': 'mysql',
        'host': 'localhost',
        'user': 'root',
        'password': '',
        'database': 'orm',
        'port': '3306',
        'prefix': '',
        'grammar': 'mysql',
        'options': {
            'charset': 'utf8mb4',
        },
    },
    'sqlite': {
        'driver': 'sqlite',
        'database': 'orm.db',
        'prefix': ''
    }
}
```

## Models

Models are also identical in terms of setup. With Orator you need to set up a the model with the database resolver like this:

```python
# Orator
DB = DatabaseManager(DATABASES)
Model.set_connection_resolver(DB)
```

And then import this `Model` class from wherever you performed this statement. With Masonite you do not need this but will instead simply import the `Model` class from Masonite:

```python
# Masonite
from masonite.orm.models import Model

class User(Model):
    pass
```

## Scopes

Scopes are also identical but the import changes:

```python
# Orator:
from orator.orm import scope

class User(Model):

    @scope
    def popular(self, query):
        return query.where('votes', '>', 100)
```

Masonite is also identical but the import changes:

```python
# Masonite:
from masonite.orm.scopes import scope

class User(Model):

    @scope
    def popular(self, query):
        return query.where('votes', '>', 100)
```

Use of scopes if identical:

```python
from user import User

# Orator:
User.popular().get()

# Masonite:
User.popular().get()
```

