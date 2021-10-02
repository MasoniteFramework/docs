Masonite ships with a "pub and sub" style events feature that allows you to subscirbe to various events and run listeners, or additional logic, when those events get emitted.

# Creating an Event

The first step in events is creating an event to listen to.

Events are simple classes that you can create wherever you like:

```python
$ python craft event UserAdded
```

This will create a simple class we can later emit

# Creating A Listener

The listener will run the logic when the event is emitted. You can create as many listeners as you like and register as many listeners to events as you need to.

To create a listener simply run the command:

```python
$ python craft listener WelcomeEmail
```

This will create a class like this:

```python
class WelcomeEmail:
    def handle(self, event):
        pass
```

## Handle Method

The handle method will run when the listener runs. It will pass the event as the first parameter and any additional arguments that are emitted from the event as additional parameters.

# Registering Events and Listeners

After your events and listeners are created you will need to register them to the event class.

You can do this via a service provider:

```python
class EventsProvider(Provider):
		def register(self):
    		self.application.make('event').listen(UserAddedEvent, [WelcomeListener])
```

# Firing Events

To fire an event you can use the `fire` method from the `Event` class:

```python
from app.events import UserAddedEvent
from masonite.events import Event

class RegisterController:
  def register(self, event: Event):
    # Register user
    event.fire(UserAddedEvent)
```

# Building a Welcome Email Listener

As an example, to build a listener that sends an email:

First, create the listener:

```python
$ python craft listener WelcomeEmail
```

Then we can build out the listener. 

To send an email we will need to import the mailable class and send the email using the `mail` key from the container:

```python
from app.mailables.WelcomeMailable import WelcomeMailable

class WelcomeEmail:
  def handle(self, event):
    from wsgi import application

    application.make("mail").send(
      WelcomeMailable().to('idmann509@gmail.com')
    )
```

You can then register the event inside the provider:

```python
class EventsProvider(Provider):
		def register(self):
    		self.application.make('event').listen(UserAddedEvent, [WelcomeListener])
```

When you emit the `UserAdded` event inside the controller, or somewhere else in the project, it will now send this email out.

You can register as many listeners to the events as you like by simply adding more listeners to the list.