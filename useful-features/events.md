# Events

## Introduction

Masonite Events is a simple to use integration for subscribing to events. These events can be placed throughout your application and triggered when various actions occur. For example you may want to do several actions when a user signs up like:

* Send an email
* Subscribe to MailChimp
* Update a section of your database

These actions can all be executed with a single line of code in you controller once you have setup your listeners

## Getting Started

### Installation

First we'll need to install the package using pip:

```text
$ pip install masonite-events
```

### Service Provider

Once installed we'll need to add the provider to our providers list:

{% code-tabs %}
{% code-tabs-item title="config/providers.py" %}
```python
from events.providers import EventProvider
...

PROVIDERS = [
    ...
    EventProvider,
    ...
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Commands

This Service Provider will add a new `event:listener` command we can use to create new events. We'll walk through step by step on how to create one below.

## Creating an Event Listener

Masonite Events allows you subscribe to arbitrary event actions with various event listeners. In this article we will walk through the steps on setting up listeners for a new user subscribing to our application.

We can create our event listener by executing the following command:

```text
$ craft event:listener SubscribeUser
```

This will create a new event in the app/events directory that looks like this:

```python
""" A SubscribeUser Event """
from events import Event

class SubscribeUser(Event):
    """ SubscribeUser Event Class """

    subscribe = []

    def __init__(self):
        """ Event Class Constructor """
        pass

    def handle(self):
        """ Event Handle Method """
        pass

```

### About the Listener class

This is a very simple class. We have the `__init__` method which is resolved by the container and we have a `handle` method, also resolved by the container.

This means that we can use syntax like:

```python
from masonite.request import Request
...
def __init__(self, request: Request):
    """ Event Class Constructor """
    self.request = request
...
```

### Subscribe Attribute

The subscribe attribute can be used as a shorthand later which we will see but it should be a list of events we want to subscribe to:

```python
from package.action import SomeAction
...
subscribe = ['user.subscribed', SomeAction]
...
```

## Subscribing to events

We can listen to events simply by passing the listener into one of your applications Service Provider's boot methods. Preferably this Service Provider should have `wsgi=False` so that you are not continuously subscribing the same listener on every request.

You likely have a Service Provider whose wsgi attribute is false but let's create a new one:

```text
$ craft provider ListenerProvider
```

Make sure we set the wsgi attribute to False:

```python
''' A ListenerProvider Service Provider '''
from masonite.provider import ServiceProvider

class ListenerProvider(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self):
        pass

```

Now we can import our listener and add it to our boot method:

```python
''' A ListenerProvider Service Provider '''
from masonite.provider import ServiceProvider
from app.events.SubscribeUser.SubscribeUser
from events import Event

class ListenerProvider(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self, event: Event):
        event.subscribe(SubsribeUser)

```

{% hint style="info" %}
This is the recommended approach over the more manual approach found below but both options are available if you find a need for one over the other.
{% endhint %}

Since we have a subscribe attribute on our listener, we can simply pass the action into the subscribe method. This will subscribe our listener to the `SomeAction` and the `user.subscribed` action that we specified in the `subscribe` attribute of our listener class.

## Manually listening to events

If we don't specify the actions in the subscribe attribute, we can manually subscribe them using the listen method in our Service Provider's boot method:

```python
''' A ListenerProvider Service Provider '''
from masonite.provider import ServiceProvider
from app.events.SubscribeUser.SubscribeUser
from events import Event

class ListenerProvider(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self, event: Event):
        event.listen('user.subscribed', [SubsribeUser])

```

{% hint style="warning" %}
Ensure that the second parameter in the listen method is a list; even if it has only a single value
{% endhint %}

## Firing events

Now that we have events that are being listened to, we can start firing events. There are two ways to fire events. We can do both in any part of our application but we will go over how to do so in a controller method.

### Fire Method

```python
from events import Event
...
def show(self, event: Event):
    event.fire('user.subscribed')
```

### Event Helper Method

Masonite Events also comes with a new builtin helper method:

```python
def show(self):
    event('user.subscribed')
```

Both of these methods will fire all our listeners that are listening to the `user.subscribed` event action. 

### Class Events

As noted briefly above, we can subscribe to classes as events:

```python
from package.action import SomeAction

def show(self):
    event(SomeAction)
```

This will go through the same steps as an event subscribed with a string above.

### Wildcard Events

We can also fire events using an \* wildcard action:

```python
def show(self):
    event('user.*')
```

This will fire events such as `user.subscribed`, `user.created`, `user.deleted`.

We can also fire events with an asterisk in the front:

```python
def show(self):
    event('*.created')
```

This will fire events such as `user.created`, `dashboard.created` and `manager.created`.

We can also fire events with a wildcard in the middle:

```python
def show(self):
    event('user.*.created')
```

This will fire events such as `user.manager.created`, `user.employee.created` and `user.friend.created`.

