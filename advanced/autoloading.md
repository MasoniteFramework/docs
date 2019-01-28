# Autoloading

## Introduction

Masonite has a [Service Container](../architectural-concepts/service-container.md) which allows you to add objects into the container and have them auto resolved in controllers and other classes. This is an excellent feature and what makes Masonite so powerful. The most obvious way to load classes into the container is through creating a [Service Provider](../architectural-concepts/service-providers.md) and interacting with the container from there.

With Masonite 2, we can use the builtin autoloader in order to load classes into the container in a much simpler way.

## Configuration

The configuration variable for autoloading is inside the `config/application.py` file which contains a list of directories:

```python
AUTOLOAD = [
    'app',
]
```

Out of the box, Masonite will autoload all classes that are located in the `app` directory which unsurprisingly contains all of the application models.

## How It Works

Masonite will go through each directory listed and convert it to a module. For example if given the directory of `app/models` it will convert that to `app.models` and fetch that module. It will use inspection to go through the entire module and extract all classes imported or defined.

If your code looks something like:

```python
from orator.orm import belongs_to
from config.database import Model

class User(Model):
    pass
```

Then the autoloader will fetch three classes: the `belongs_to` class, the `Model` class and the `User` class. The autoloader will then check if the module of the classes fetched are actually apart of the module being autoloaded.

In other words the modules of the above classes are: `orator.orm`, `config.database` and `app` respectively. Remember that we are just autoloaded the `app` module so it will only bind the `app.User` class to the container with a binding of the class name: `User` and the actual object itself: `<class app.User.User>`.

All of this autoloading is done when the server is first started but before the WSGI server is ready to start accepting requests so there are no performance hits for this.

## Usage

Since the app directory is autoloaded, and our `User` model is in that directory, the `User` model will be loaded into the container when the server starts.

All bindings into the Service Container will be the name of the object as the key and the actual object as the value. So the `User` model will be accessed like:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
from app.User import User

def show(self, user: User):
    user.find(1)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

There is no reason to add models into the container unless your specific use case requires it but this is just an example.

### Other Directories

You don't have to keep your models in the `app` directory. Feel free to move them anywhere but they will not be autoloaded outside the app directory by default. In order to autoload other directories we can add them to the `AUTOLOAD` variable.

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
from app.User import User
from app.Blog import Blog
from app.Author import Author

def show(self, user: User, blog: Blog, author: Author):
    user.find(1)
    blog.find(1)
    author.find(1)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
Being that the container is useful as an IOC container, another use case would be if a third party library needed some models to manipulate and then bind them back into the container. An example of this type of library would be one that needs to change the models methods in order to capture query operations and send them to a dashboard or a report.
{% endhint %}

## Exceptions

The autoload class will raise a few exceptions so you should be aware of them in order to avoid confusion when these exceptions are raised.

### InvalidAutoloadPath

This exception will be thrown if any of your autoload paths end with a forward slash like the first element in this list:

```python
AUTOLOAD = [
    'app/',
    'app/models'
]
```

Notice the path is now app/ and not app. This will throw an exception when the server first starts.

### AutoloadContainerOverwrite

This exception will be thrown when one of your classes are about to overwrite a container binding that is outside of your search path. The search path being the directories you specified in the `AUTOLOAD` constant.

For example, you may have a model called `Request` like so:

```text
app/
  http/
  providers/
  User.py
  Request.py
bootstrap/
...
```

Without this exception, your application will overwrite the binding of the Masonite `Request` class.

When Masonite goes to autoload these classes, it will detect that the `Request` key has already been bound into the container \(by Masonite itself\). Masonite will then detect if that `Request` object in the container is within the search path. In other words it will check for a `Request` class inside the current module you are autoloading.

If the object is outside of the module you are autoloading then it will throw this exception. In this instance, it will throw an exception because the `Request` key in the container is the &lt;class masonite.request.Request&gt; class which is outside of the `app` module \(and inside the `masonite.request` module\).

If you find yourself hitting this exception then move the object outside of a directory being autoloaded and into a separate directory outside of the autoloader and then you can manually bind into the container with a different key, or simply rename the class to something else. When using models, you can rename the model to whatever you like and then specify a `__table__` attribute to connect the model to the specific table.

## Annotations

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

You may also want to autoload classes yourself. This may be useful if building a package and needing to get all classes from a certain location or even all instances from a certain directory.

### Class Instances

This might in useful in the Masonite scheduling feature to grab all the classes that are subclasses of the `Task` class.

Autoloading class instances could look something like:

```python
from masonite.autoload import Autoload
from orator.orm import Model

classes = Autoload().instances(['app/models'], Model)
# returns {'Model': <config.database.Model>, 'User': <class app.models.User.User>, ...}
```

This will fetch all the classes in the `app/models` directory that are instances of the `Model` class. The `classes` attribute contains a dictionary of all the classes it found.

### Class Collecting

If you don't want to get classes that are instances of another class then we can simply collect classes:

```python
from masonite.autoload import Autoload
from orator.orm import Model

classes = Autoload().collect(['app/models'])
# returns {'Model': <config.database.Model>, 'User': <class app.models.User.User>, ...}
```

### Getting Only Application Classes

You may have noticed that this autoload class will actually capture all classes found. These classes include the classes that were imported as well.

{% hint style="info" %}
The default autoload in the config file only loads application specific classes.
{% endhint %}

We can only get application specific classes by passing a parameter in:

```python
from masonite.autoload import Autoload
from orator.orm import Model

classes = Autoload().collect(['app/models'], only_app=True)
# returns {'User': <class app.models.User.User>, ...}
```

This will check the namespace on the class found and make sure it is apart of the path that is being searched.

### Instantiating Classes

The classes that the autoloader finds are just classes themselves which are uninstantiated. This is great for fetching models but won't be good for fetching other objects like commands. Commands need to be instantiated when they go into the container by design so they can be found by Masonite and cleo.

We can tell the autoloader to instantiate the class by passing another parameter. This will simply instantiate the object without passing any parameters in.

{% hint style="info" %}
If your class needs parameters then you should collect the classes and instantiate each one individually.
{% endhint %}

```python
from masonite.autoload import Autoload
from orator.orm import Model

classes = Autoload().collect(['app/models'], only_app=True, instantiate=True)
# returns {'User': <app.models.User.User at 0x10e36d780>, ...}
```

You can see that now all the objects found are instantiated.

