# Controllers

## Introduction

Controllers are a vital part of Masonite and is mainly what differs it from other Python frameworks that implement the MVC structure differently. Controllers are simply classes with methods. These methods take a `self` parameter which is the normal self that Python class methods require. Controller methods can be looked at as function based views if you are coming from Django as they are simply methods inside a class and work in similar ways.

Controllers have an added benefit over straight function based views as the developer has access to to a full class they can manipulate however they want. In other words, controller methods may utilize class attributes or private methods to break up logic. They provide a lot of flexibility.

## Defining a Controller

Its very easy to create a controller with Masonite with the help of our `craft` command tool. We can simply create a new file inside `app/http/controllers`, name the class the same name as the file and then create a class with methods. We can also use the `craft controller` command to do all of that for us which is:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller DashboardController
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When we run this command we now have a new class in `app/http/controllers/DashboardController.py` called `DashboardController`. By convention, Masonite expects that all controllers have their own file since it’s an extremely easy way to keep track of all your classes since the class name is the same name as the file but you can obviously name this class wherever you like.

## Defining a Controller Method

Controller methods are very similar to function based views in a Django application except this is just a normal class method. Our controller methods at a minimum should look like:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
def show(self):
    pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

All controller methods must have the self parameter. The `self` parameter is the normal python `self` object which is just an instance of the current class as usual. Nothing special here.

All controller methods are resolved by the container so you may also retrieve additional objects from the container by specifying them as a parameter:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/DashboardController.py" %}
```python
def show(self, Request):
    print(Request) # Grabbed the Request object from the container
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="warning" %}
**This might look magical to you so be sure the read about the IOC container in the** [**Service Container**](../architectural-concepts/service-container.md) **documentation.**
{% endhint %}

It’s important to note that unlike other frameworks, we do not have to specify our route parameters as parameters in our controller method. We can retrieve the parameters using the `Request.param('key')` class method.

{% hint style="success" %}
Read about how to create and use views by reading the [Views ](views.md)documentation
{% endhint %}

