# Autoloading

## Introduction

Masonite has a [Service Container](../architectural-concepts/service-container.md) which allows you to add objects into the container and have them auto resolved in controllers and other classes. This is an excellent feature and what makes Masonite so powerful. The most obvious way to load classes into the container is through creating a [Service Provider](../architectural-concepts/service-providers.md) and interacting with the container from there.

With Masonite 2, we can use the builtin autoloader in order to load classes into the container in another way.

## Configuration

The configuration variable for autoloading is inside the `config/application.py` file which contains directories:

```python
AUTOLOAD = [
    'app',
]
```

Out of the box, Masonite will autoload all classes that are located in the app directory which unsuprisingly contains all of the application models.

## Usage

Since the app directory is autoloaded, and our User model is in that directory, the User model will be loaded into the container when the server starts.

All bindings into the Service Container will be the name of the object as the key and the actual object as the value. So the User model will be accessed like:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self, User):
    User.find(1)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Other Directories

You don't have to keep your models in the app directory. Feel free to move them anywhere but they will not be autoloaded outside the app directory by default. In order to autoload other directories we can add them to the `AUTOLOAD` variable.

For example if we have an app directory structure like:

```text
app/
  http/
  providers/
  models/
    Blog.py
    Author.py
  User.py
```

Then we can edit our `AUTOLOAD` variable like so:

```python
AUTOLOAD = [
    'app',
    'app/models'
]
```

And then be able to do:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self, User, Blog, Author):
    User.find(1)
    Blog.find(1)
    Author.find(1)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
Being that the container is useful as an IOC container, another use case would be if a third party library needed some models to manipulate and then bind them back into the container. An example of this type of library would be one that needs to change the models methods in order to capture query operations and send them to a dashboard or a report.
{% endhint %}

### Annotations

Although it is useful to get the model by the actual container key name, it might not be as practical or even the best way to fetch models from the container. 

The recommended approach is to simply fetch the class itself by using annotations so you can adjust the variable name and ensure consistency throughout your application.

```python
from app.User import User

def show(self, author: User):
    author.find(1)
```

{% hint style="info" %}
If this seems like a strange syntax to you, be sure to read the [Resolve](../architectural-concepts/service-container.md#resolve) section of the [Service Container](../architectural-concepts/service-container.md) documentation.
{% endhint %}

## Autoload Class

You may also want to autoload classes yourself. This may be useful if building a package and needing to get all classes from a certain location or even all instances from a certain directory. For example this might in useful in the Masonite scheduling feature to grab all the classes that are subclasses of the `Task` class.

This may look something like:

```python
from masonite.autoload import Autoload
from orator.orm import Model

autoload = Autoload().instances(['app/models'], Model)
autoload.classes # returns {'User': <class app.User.User>, ...}
```

This will fetch all the classes in the `app/models` directory that are instances of the `Model` class. The `classes` attribute contains a dictionary of all the classes it found.

