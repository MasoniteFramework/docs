Masonite comes with a powerful way to broadcast events in your application. These events can be listened to on the frontend using JavaScript/Vue and the Pusher library. These can be things like a new notification which you can show on the frontend without reloading the page.

# Broadcasting

You can broadcast a dictionary onto any number of channels:

```python
from masonite.broadcasting import Broadcast
from app.broadcasts import UserAdded

def emit(self, broadcast: Broadcast)
	broadcast.channel('channel_name', "event_name", {"key": "value"})
```

This will emit an event to all  applications listening to this public event.

You may broadcast on multiple channels as well:

```python
broadcast.channel(['channel1', 'channel2'], "event_name", {"key": "value"})
```

> This type of broadcasting will emit all channels as public. For private and presence channels, keep reading.

# Creating a Broadcasting Event

Broadcasting events are simple classes that inherit from `CanBroadcast`. You may use any class, including the Masonite Event classes.

A broadcasting event will look like this:

```python
from masonite.broadcasting import CanBroadcast, Channel

class UserAdded(CanBroadcast):
  
  # ..

	def broadcast_on(self):
  	return Channel("channel_name")
```

# Broadcasting The Event

You can broadcast the event using the `Broadcast` class:

```python
from masonite.broadcasting import Broadcast
from app.broadcasts import UserAdded

def emit(self, broadcast: Broadcast)
	broadcast.channel(UserAdded())
```

# Channel Options

You can broadcast on any numbers of channels. 

## Public Channels

Inside the event class you can specify a public channel. These channels allow anyone with a connection to listen to events on this channel:

```python
from masonite.broadcasting import Channel

def broadcast_on(self):
  return Channel("channel_name")
```

## Private Channels

Private channels require authorization from the user to connect to the channel. You can use this channel to emit only events that a user should listen in on.

Private channels are channels that start with a `private-` prefix. When using private channels, the prefix will be prepended for you automatically.

Private channels can only be broadasting on if users are logged in. When the channel is authorized, it will check if the user is currently authenticated before it broadcasts. If the user is not authenticated it will not broadcast anything on this channel.

```python
from masonite.broadcasting import PrivateChannel

def broadcast_on(self):
  return PrivateChannel("channel_name")
```

This will emit events on the `private-channel_name` channel.

### Routing

On the frontend, when you make a connection to a private channel, a POST requets gets made to authenticate the private channel. Masonite ships with this authentication route for you. All you need to do is add it to your routes:

```python
from masonite.broadcasting import Broadcast

ROUTES = [
  # Normal route list here
]

ROUTES += Broadcast.routes()
```

This will create a route you can authenticate you private channel on the frontend. The authorization route will be `/broadcasting/authorize` but you can change this to anything you like:

```python
ROUTES += Broadcast.routes(auth_route="/pusher/auth")
```

You will also need to add the `/pusher/auth` route to the CSRF exemption:

```python
class VerifyCsrfToken(Middleware):

    exempt = [
        '/pusher/auth'
    ]
```

## Presence Channels

Presence channels work exactly the same as private channels except you can see who else is inside this channel. This is great for chatroom type applications.

For Presence channels, the user also has to be authenticated.

```python
from masonite.broadcasting import PresenceChannel

def broadcast_on(self):
  return PresenceChannel("channel_name")
```

This will emit events on the `presence-channel_name` channel.

### Routing

Adding the authentication route is also the exact same as private channels:

```python
from masonite.broadcasting import Broadcast

ROUTES = [
  # Normal route list here
]

ROUTES += Broadcast.routes()
```

This will create a route you can authenticate you private channel on the frontend. The authorization route will be `/broadcasting/authorize` but you can change this to anything you like:

```python
ROUTES += Broadcast.routes(auth_route="/pusher/auth")
```
