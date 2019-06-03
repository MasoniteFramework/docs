# Controllers

## Introduction

Controllers are a vital part of Masonite and is mainly what differs it from other Python frameworks which all implement the MVC structure differently. Controllers are simply classes with methods. These methods take a `self` parameter which is the normal self that Python methods require. Controller methods can be looked at as "function based views" if you are coming from Django as they are simply methods inside a class and work in similar ways.

Controllers have an added benefit over straight function based views as the developer has access to a full class they can manipulate however they want. In other words, controller methods may utilize class attributes, private methods and class constructors to break up and abstract logic. They provide a lot of flexibility.

## Creating a Controller

Its very easy to create a controller with Masonite with the help of our `craft` command tool. We can simply create a new file inside `app/http/controllers`, name the class the same name as the file and then create a class with methods. We can also use the `craft controller` command to do all of that for us which is:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller Dashboard
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When we run this command we now have a new class in `app/http/controllers/DashboardController.py` called `DashboardController`. By convention, Masonite expects that all controllers have their own file since it’s an extremely easy way to keep track of all your classes since the class name is the same name as the file. This is very opionated but you can obviously put this class wherever you like.

{% hint style="info" %}
Notice that we passed in `Dashboard` but created a `DashboardController`. Masonite will always assume you want to append `Controller` to the end.
{% endhint %}

### Exact Controllers

Remember that Masonite will automatically append Controller to the end of all controllers. If you want to create the exact name of the controller then you can pass a `-e` or `--exact` flag.

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller Dashboard -e
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller Dashboard --exact
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create a `Dashboard` controller located in `app/http/controllers/Dashboard.py`

### Resource Controllers

Resource controllers are controllers that have basic CRUD / resource style methods to them such as create, update, show, store etc. We can create a resource controller by running:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller Dashboard -r
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller Dashboard --resource
```
{% endcode-tabs-item %}
{% endcode-tabs %}

this will create a controller that looks like:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
""" A Module Description """

class DashboardController: 
 """Class Docstring Description
 """

    def show(self): 
        pass

    def index(self): 
        pass

    def create(self): 
        pass

    def store(self): 
        pass

    def edit(self): 
        pass

    def update(self): 
        pass

    def destroy(self): 
        pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Defining a Controller Method

Controller methods are very similar to function based views in a Django application. Our controller methods at a minimum should look like:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
def show(self):
    pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If you are new to Python, all controller methods must have the self parameter. The `self` parameter is the normal python `self` object which is just an instance of the current class as usual. Nothing special here.

## Container Resolving

All controller methods and constructors are resolved by the container so you may also retrieve additional objects from the container by specifying them as a parameter in the method:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
from masonite.request import Request
...

def show(self, request: Request):
    print(request) # Grabbed the Request object from the container
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or by specifying them in the constructor:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
from masonite.request import Request

class DashboardController:

    def __init__(self, request: Request):
        self.request = request

    def show(self):
        print(self.request) # Grabbed the Request object from the container
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If you need a class in multiple controller methods then it is recommended to put it into the constructor in order to keep the controller DRY.

{% hint style="warning" %}
**This might look magical to you so be sure to read about the IOC container in the** [**Service Container**](../architectural-concepts/service-container.md) **documentation.**
{% endhint %}

It’s important to note that unlike other frameworks, we do not have to specify our route parameters as parameters in our controller method. We can retrieve the parameters using the `request.param('key')` class method.

{% hint style="success" %}
Read about how to create and use views by reading the [Views ](views.md)documentation
{% endhint %}

## Returning JSON

You can return JSON in a few different ways. The first way is returning a dictionary which will then be parsed to JSON:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
def show(self):
    return {'key': 'value'}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

you may return a list:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
def show(self):
    return ['key', 'value']
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Or you may even return a model instance or collection. Take these 2 code snippets as an example:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
from app.User import User

def show(self):
    return User.find(1)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
from app.User import User

def show(self):
    return User.where('active', 1).get()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Passing Route Parameters

Optionally you can pass route parameters along with your resolving code. This is useful to keep a nice clean codebase. 

For example, these two code snippets are the same:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
from masonite.request import Request
from masonite.view import View
...

def show(self, request: Request, view: View):
    return User.find(request.param('user_id'))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

And this:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
from masonite.view import View
...

def show(self, user_id, view: View):
    return User.find(user_id)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can specify parameters along with any other container resolving.

