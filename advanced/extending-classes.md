# Extending Classes

## Introduction

It's common to want to use a Service Provider to add new methods to a class. For example, you may want to add a `is_authenticated` method to the `Request` class. Your package and Service Provider may be for a better authentication system.

You may easily extend classes that inherit from the `Extendable` class. Many of the built in classes inherit from it.

## Usage

You have a few options for adding methods to any of the core classes. You can extend a class with functions, classes and class methods. Typical usage may look like:

```python
def is_authenticated(self):
    return self

def show(self, Request):

    Request.extend(is_authenticated)

    print(Request.is_authenticated()) # returns the Request class
```

Usage is very simple and has several options for extending a class. Notice that we don't call the function but we pass the reference to it.

### Extending a function

This will simply add the function as a bound method to the `Request` class

```python
def is_authenticated(self):
    return self

def show(self, Request):

    Request.extend(is_authenticated)

    print(Request.is_authenticated()) # returns the Request class
```

### Extending a class method

We can also extend a class method which will take the method given and add it as a bound method.

```python
class Authentication:

    def is_authenticated(self):
        return self

def show(self, Request):

    Request.extend(Authentication.is_authenticated)

    print(Request.is_authenticated()) # returns the Request class
```

### Extending a class

We can even extend a whole class which will get all the classes methods and create bound methods to the Request class.

```python
class Authentication:

    def is_authenticated(self):
        return self

    def login(self):
        return self

def show(self, Request):

    Request.extend(Authentication)

    print(Request.is_authenticated()) # returns the Request class
    print(Request.login()) # returns the Request class
```

