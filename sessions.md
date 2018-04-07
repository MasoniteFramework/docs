# Sessions

# Introduction

You'll find yourself needing to add temporary data to an individual user. Sessions allow you to do this by adding a key value pair and attaching it to the user's IP address. You can reset the session data anytime you want; which is usually done on logout or login actions.

The Session features are adding to the framework though the `SessionProvider` Service Provider.

## Getting Started

There are a two ideas behind sessions. There is **session data** and **flash data**. Session data is any data that is persistent for the duration of the session and flash data is data that is only persisted on the next request.

## Using Sessions

Sessions is loaded into the container with the `Session` key. So you may access the `Session` class in any parts of the code that is resolved by the container. These include controllers, drivers, middleware etc:

```python
def show(self, Session):
    print(Session) # Session class
```

## Setting Data

Data can easily be persisted to the current user by using the `set` method.

```python
def show(self, Session):
    Session.set('key', 'value')
```

This will update a dictionary that is linked to the current user.

## Getting Data

Data can be pulled from the session:

```python
def show(self, Session):
    Session.get('key') # Returns 'value'
```

## Checking Data

Very often, you will want to see if a value exists in the session:

```python
def show(self, Session):
    Session.has('key') # Returns True
```

## Getting all Data

You can get all data for the current user:

```python
def show(self, Session):
    Session.all() # Returns {'key': 'value'}
```

## Flashing Data

Data can be inserted only for the next request. This is useful when using redirection and displaying a success message.

```python
def show(self, Session):
    Session.flash('success', 'Your action is successful')
```

## Templates

The `SessionProvider` comes with a helper method that is automatically attached to all templates. You can use the session helper just like you would use the `Session` class.

```python
{% if session().has('success') %}
    <div class="alert alert-success">
        {{ session().get('success')
    </div>
{% endif %}
```







