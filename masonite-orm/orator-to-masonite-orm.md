# Orator To Masonite ORM

## Masonite To Orator Upgrade Guide



This guide will explain how to move from Orator to Masonite ORM. Masonite ORM was made to be pretty much a straight port of Orator but allow the Masonite organization complete creative control of the ORM.



Before moving your project over to Masonite ORM please keep in mind some features are not *\(*at least currently*\)* ported over from Orator. These are features that may be ported over in the future. These features are:



* Pagination

* Inserting related models

* Pessimistic Locking

* Model Events and Observers

* `.change()` functionality in migrations

* through relationships



## Config



The configuration dictionary between Orator and Masonite ORM is identical. The only difference is that Masonite ORM ***\*****requires*****\*** a `config/database.py` file whereas Orator was optional and needed to be explicitly specified in several places like commands.

If you are coming from Masonite already then don't worry, this file is already there. If not you will need to create this `config/database.py` file.

This is an example of a Masonite ORM config dictionary:

## Models

Models are identical but the imports are different. Orator requires you to set the model resolver and then you import that model.

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
# Orator
from masoniteorm.scopes import scope

class User(Model):
  
  @scope
  def popular(self, query):
		return query.where('votes', '>', 100)
```
