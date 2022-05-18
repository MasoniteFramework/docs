Masonite comes with a powerful way to broadcast events in your application. These events can be listened to client-side using Javascript.

These can be things like a new notification which you can show on the frontend without reloading the page.

Masonite comes with one server-side driver: [Pusher](https://pusher.com/channels).

# Configuration

## Server Side

Server side configuration for broadcasting is done in `config/broadcast.py` configuration file. For now there is one driver available [Pusher](https://pusher.com/channels).

You should create an account on Pusher Channels and then create a Pusher application on your account and get the related credentials (client, app_id, secret) and the cluster location name and put this into the broadcast pusher options.

```python
"pusher": {
    "driver": "pusher",
    "app_id": "3456678"
    "client": "478b45309560f3456211" # key
    "secret": "ab4229346et64aa8908"
    "cluster": "eu",
    "ssl": False,
},
```

Finally make sure you install the `pusher` python package

```shell
pip install pusher
```

## Client Side

To be able to receive broadcast events in the browser you should install [Javascript Pusher SDK](https://pusher.com/docs/channels/getting_started/javascript).

Include the pusher-js script tag on your page

```html
<script src="https://js.pusher.com/7.0.3/pusher.min.js"></script>
```
{% hint style="warning" %}
This is the quickest way to install Pusher. But for a real app you will often use a build system to [install and compile assets](/features/compiling-assets.md#compiling) and will install the Javascript Pusher SDK with `npm install pusher-js` and then import Pusher class with `import Pusher from 'pusher-js';`.
{% endhint %}

Create a Pusher instance configured with your credentials

```javascript
const pusher = new Pusher("478b45309560f3456211", {
  cluster: "eu",
});
```

{% hint style="warning" %}
It is advised to use environment variables instead of hard-coding credentials client-side. If you're using Laravel Mix to [compile assets](/features/compiling-assets.md#compiling) then you should prefix your environment variables with `MIX_`.
{% endhint %}

Now you're ready to subscribe to channels and listen for channels events.


# Creating Events

Broadcast events are simple classes that inherit from `CanBroadcast`. You may use any class, including the Masonite Event classes.

A broadcast event will look like this:

```python
from masonite.broadcasting import CanBroadcast, Channel

class UserAdded(CanBroadcast):

    def broadcast_on(self):
        return Channel("channel_name")
```

# Broadcasting Events

You can broadcast the event using the `Broadcast` facade or by resolving `Broadcast` class from container.

You can broadcast easily without creating a Broadcast event

```python
from masonite.facades import Broadcast

Broadcast.channel('channel_name', "event_name", {"key": "value"})
```

Or you can broadcast the event class created earlier

```python
from masonite.facades import Broadcast
from app.broadcasts import UserAdded

broadcast.channel(UserAdded())
```

You may broadcast on multiple channels as well:

```python
Broadcast.channel(['channel1', 'channel2'], "event_name", {"key": "value"})
```

{% hint style="info" %}
This type of broadcasting will emit all channels as public. For private and presence channels, keep reading.
{% endhint %}


# Listening For Events

In this section we will use the client `pusher` instance [configured earlier](#client-side).

To listen for events on client-side you must first subscribe to the channel the events are emitted on

```javascript
const channel = pusher.subscribe("my-channel");
```

Then you can listen for events

```javascript
channel.bind("my-event", (data) => {
  // Method to be dispatched when event is received
});
```

# Channel Types

Different channel types are included in Masonite.

## Public Channels

Inside the event class you can specify a Public channel. These channels allow anyone with a connection to listen to events on this channel:

```python
from masonite.broadcasting import Channel

class UserAdded(CanBroadcast):

    def broadcast_on(self):
        return Channel("channel_name")
```

## Private Channels

Private channels require authorization from the user to connect to the channel. You can use this channel to emit only events that a user should listen in on.

Private channels are channels that start with a `private-` prefix. When using private channels, the prefix will be prepended for you automatically.

Private channels can only be broadasting on if users are logged in. When the channel is authorized, it will check if the user is currently authenticated before it broadcasts. If the user is not authenticated it will not broadcast anything on this channel.

```python
from masonite.broadcasting import PrivateChannel

class UserAdded(CanBroadcast):

    def broadcast_on(self):
      return PrivateChannel("channel_name")
```

This will emit events on the `private-channel_name` channel.

### Routing

On the frontend, when you make a connection to a private channel, a POST request is triggered by the broadcast client to authenticate the private channel. Masonite ships with this authentication route for you. All you need to do is add it to your routes:

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

You will also need to add the `/pusher/auth` route to the CSRF exemption.

```python
class VerifyCsrfToken(Middleware):

    exempt = [
        '/pusher/auth'
    ]
```
The reason for this is that the broadcast client will not send the CSRF token along with the POST authorization request.

## Presence Channels

Presence channels work exactly the same as private channels except you can see who else is inside this channel. This is great for chatroom type applications.

For Presence channels, the user also has to be authenticated.

```python
from masonite.broadcasting import PresenceChannel

class UserAdded(CanBroadcast):

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

# Examples

To get started more easily with event broadcasting in Masonite, two small examples are available here:
- Sending public app releases notification to every users (using [Public channels](#public-channels))
- Sending private user notifications to the logged in users (using [Private channels](#private-channels))

## Sending public app releases notification

Examples are coming soon.

## Sending private user notifications

Examples are coming soon.
