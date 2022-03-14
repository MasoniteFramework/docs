# Notifications

Masonite has a simple yet powerful notification feature which is used to send notifications from your application. Here is a brief overview of what you can do with notifications:

* Send `E-mail`, `Slack` and `SMS` notifications
* Store notifications in a database so they may be displayed in your web interface.
* Queue notifications
* Broadcast notifications

## Creating a Notification

To create and send a notification with Masonite, you must first build a `Notification` class. This class will act as the settings for your notification such as the delivery channels and the content of the notification for those different channels (mail, slack, sms, ...).

The first step of building a notification is running the command:

```
$ python craft notification Welcome
```

This will create your notification and it will look something like this:

```python
class Welcome(Notification, Mailable):

    def to_mail(self, notifiable):
        return (
            self.to(notifiable.email)
            .subject("Welcome to our site!")
            .from_("admin@example.com")
            .text(f"Hello {notifiable.name}")
        )

    def via(self, notifiable):
        return ["mail"]
```

Each notification class has a `via` method that specify on which channels the notification will be delivered. Notifications may be sent on the `mail`, `database`, `broadcast`, `slack` and `vonage` channels. More details on this later. When sending the notification it will be automatically sent to each channel.

{% hint style="info" %}
If you would like to use an other delivery channel, feel free to check if a community driver has been developed for it or [create your own driver and share it with the community](notifications.md) !
{% endhint %}

`via` method should returns a list of the channels you want your notification to be delivered on. This method receives a `notifiable` instance.

```python
class WelcomeNotification(Notification, Mailable):
    ...
    def via(self, notifiable):
        """Send this notification by email, Slack and save it in database."""
        return ["mail", "slack", "database"]
```

## Sending a Notification

### Basic

You can send your notification inside your controller easily by using the `Notification` class:

```python
from masonite.notification import NotificationManager
from app.notifications.Welcome import Welcome

class WelcomeController(Controller):

    def welcome(self, notification: NotificationManager):
        notification.route('mail', 'sam@masonite.com').send(Welcome())
```

If the notification needs to be delivered to multiple channels you can chain the different routes:

```python
notification.route('mail', 'sam@masonite.com').route('slack', '#general').send(Welcome())
```

{% hint style="warning" %}
`database` channel cannot be used with those notifications because no Notifiable entity is attached to it.
{% endhint %}

### To notifiables

If you want to send notifications e.g. to your users inside your application, you can [define them as notifiables](notifications.md#using-notifiables). Then you can still use `Notification` class to send notification:

```python
from masonite.notification import NotificationManager
from app.notifications.Welcome import Welcome

class WelcomeController(Controller):

    def welcome(self, notification: NotificationManager):
        user = self.request.user()
        notification.send(user, Welcome())

        # send to all users
        users = User.all()
        notification.send(users, Welcome())
```

Or you can use a handy `notify()` method:

```python
user = self.request.user()
user.notify(Welcome())
```

## Using Notifiables

ORM Models can be defined as `Notifiable` to allow notifications to be sent to them. The most common use case is to set `User` model as `Notifiable` as we often need to send notifications to users.

To set a Model as Notifiable, just add the `Notifiable` mixin to it:

```python
from masonite.notification import Notifiable

class User(Model, Notifiable):
    # ...
```

You can now send notifications to it with `user.notify()` method.

### Routing

Then you can define how notifications should be routed for the different channels. This is always done by defining a `route_notification_for_{driver}` method.

For example, with `mail` driver you can define:

```python
from masonite.notification import Notifiable

class User(Model, Notifiable):
    ...
    def route_notification_for_mail(self):
        return self.email
```

This is actually the default behaviour of the mail driver so you won't have to write that but you can customize it to your needs if your User model don't have `email` field or if you want to add some logic to get the recipient email.

## Queueing Notifications

If you would like to queue the notification then you just need to inherit the `Queueable` class and it will automatically send your notifications into the queue to be processed later. This is a great way to speed up your application:

```python
from masonite.notification import Notification
from masonite.queues import Queueable

class Welcome(Notification, Queueable):
    # ...
```

## Channels

### Mail

You should define a `to_mail` method on the notification class to specify how to build the email notification content.

```python
class Welcome(Notification, Mailable):

    def to_mail(self, notifiable):
        return (
            self.to(notifiable.email)
            .subject("Welcome to our site!")
            .from_("admin@example.com")
            .text(f"Hello {notifiable.name}")
        )

    def via(self, notifiable):
        return ["mail"]
```

The notification will be sent using the default mail driver defined in `config/mail.py`. For more information about options to build mail notifications,
please check out [Mailable options](/features/mail.md#mail-options).

If you want to override the mail driver for a given notification you can do:

```python
class Welcome(Notification, Mailable):

    def to_mail(self, notifiable):
        return (
            self.to(notifiable.email)
            .subject("Welcome to our site!")
            .from_("admin@example.com")
            .text(f"Hello {notifiable.name}")
            .driver("mailgun")
        )

    def via(self, notifiable):
        return ["mail"]
```

### Slack

You should define a `to_slack` method on the notification class to specify how to build the slack notification content.

```python
from masonite.notification.components import SlackComponent

class Welcome(Notification):

    def to_slack(self, notifiable):
        return SlackComponent().text('Masonite Notification: Read The Docs!, https://docs.masoniteproject.com/') \
            .channel('#bot') \
            .as_user('Masonite Bot') \

    def via(self, notifiable):
        return ["slack"]
```

#### Options

`SlackComponent` takes different options to configure your notification:

| Method               | Description                                                                                                                                                                                                                                                                  | Example                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| .text()              | The text you want to show in the message                                                                                                                                                                                                                                     | .text('Welcome to Masonite!')                                              |
| .to()                | The channel you want to broadcast to. If the value you supply starts with a # sign then Notifications will make a POST request with your token to the Slack channel list API and get the channel ID. You can specify the channel ID directly if you don't want this behavior | .to('#general') .to('CHSUU862')                                            |
| .send\_from()        | The username you want to show as the message sender. You can also specify either the `url` or `icon` that will be displayed as the sender.                                                                                                                                   | .send\_from('Masonite Bot', icon=":ghost:")                                |
| .as\_current\_user() | This sets a boolean value to True on whether the message should show as the currently authenticated user.                                                                                                                                                                    | .as\_current\_user()                                                       |
| .link\_names()       | This enables finding and linking channel names and usernames in message.                                                                                                                                                                                                     | .link\_names()                                                             |
| .can\_reply()        | This auhtorizes replying back to the message.                                                                                                                                                                                                                                | .can\_reply()                                                              |
| .without\_markdown() | This will not parse any markdown in the message. This is a boolean value and requires no parameters.                                                                                                                                                                         | .without\_markdown()                                                       |
| .unfurl\_links()     | This enable showing message attachments and links previews. Usually slack will show an icon of the website when posting a link. This enables that feature for the current message.                                                                                           | .unfurl\_links()                                                           |
| .as\_snippet()       | Used to post the current message as a snippet instead of as a normal message. This option has 3 keyword arguments. The `file_type`, `name`, and `title`. This uses a different API endpoint so some previous methods may not be used.                                        | .as\_snippet(file\_type='python', name='snippet', title='Awesome Snippet') |
| .token()             | Override the globally configured token.                                                                                                                                                                                                                                      | .token('xoxp-359926262626-35...')                                          |

Notifications can be sent to a Slack workspace in two ways in Masonite:

* Slack Incoming Webhooks [more here](https://api.slack.com/messaging/webhooks)
* Slack Web API [more here](https://api.slack.com/methods/chat.postMessage)

**Incoming Webhooks**

You will need to [configure an "Incoming Webhook"](https://masoniteproject.slack.com/apps/A0F7XDUAZ-incoming-webhooks) integration for your Slack workspace. This integration will provide you with a URL you may use when routing Slack notifications. This URL will target a specific Slack channel.

**Web API**

You will need to [generate a token](https://api.slack.com/web#slack-web-api\_\_authentication) to interact with your Slack workspace.

{% hint style="info" %}
This token should have at minimum the `channels:read`, `chat:write:bot`, `chat:write:user` and `files:write:user` permission scopes. If your token does not have these scopes then parts of this feature will not work.
{% endhint %}

Then you can define this token globally in `config/notifications.py` file as `SLACK_TOKEN` environment variable. Or you can configure different tokens (with eventually different scopes) per notifications.

#### Advanced Formatting

Slack notifications can use [Slack Blocks Kit](https://api.slack.com/block-kit/building) to build more complex notifications. Before using this you just have to install `slackblocks` python API to handle Block Kit formatting.

```
$ pip install slackblocks
```

Then you can import most of the blocks available in Slack documentation and start building your notification. You need to use the `block()` option. Once again you can chain as many blocks as you want.

```python
from masonite.notification.components import SlackComponent
from slackblocks import HeaderBlock, ImageBlock, DividerBlock

class Welcome(Notification):
    def to_slack(self, notifiable):
        return SlackComponent() \
            .text('Notification text') \
            .channel('#bot') \
            .block(HeaderBlock("Header title")) \
            .block(DividerBlock()) \
            .block(ImageBlock("https://path/to/image", "Alt image text", "Image title"))
```

You can find all blocks name and options in [`slackblocks` documentation](https://github.com/nicklambourne/slackblocks#usage) and more information in [Slack blocks list](https://api.slack.com/reference/block-kit/blocks).

{% hint style="warning" %}
Some blocks or elements might not be yet available in `slackblocks`, but most of them should be there.
{% endhint %}

#### Routing to notifiable

You should define the related `route_notification_for_slack` method on your notifiable to return either

* a webhook URL or a list of webhook URLs (if you're using [Incoming Webhooks](../#slack-incoming-webhooks))
* a channel name/ID or a list of channels names/IDs (if you're using [Slack Web API](../#slack-web-api))

```python
class User(Model, Notifiable):

    def route_notification_for_slack(self, notification):
        """Examples for Incoming Webhooks."""
        # one webhook
        return "https://hooks.slack.com/services/..."
        # multiple webhooks
        return ["https://hooks.slack.com/services/...", "https://hooks.slack.com/services/..."]
```

```python
class User(Model, Notifiable):

    def route_notification_for_slack(self, notification):
        """Examples for Slack Web API."""
        # one channel name
        return "#general"
        # multiple channel name
        return ["#users", "#general"]
        # one channel ID
        return "C1234567890"
```

#### Routing to anonymous

To send a Slack notification without having a notifiable entity you must use the `route` method

```python
notification.route("slack", "#general").notify(Welcome())
```

The second parameter can be a `channel name`, a `channel ID`or a `webhook URL`.

{% hint style="warning" %}
When specifying channel names you must keep `#` in the name as in the example. Based on this name a reverse lookup will be made to find the corresponding Slack channel ID. If you want to avoid this extra call, you can get the channel ID in your Slack workspace (right click on a Slack channel > Copy Name > the ID is at the end of url)
{% endhint %}

### SMS

Sending SMS notifications in Masonite is powered by [Vonage](https://www.vonage.com/communications-apis/sms/) (formerly Nexmo). Before you can send notifications via Vonage, you need to install the `vonage` Python client.

```
$ pip install vonage
```

Then you should configure the `VONAGE_KEY` and `VONAGE_SECRET` credentials in `notifications.py` configuration file:

```python
# config/notifications.py

VONAGE = {
  "key": env("VONAGE_KEY"),
  "secret": env("VONAGE_SECRET"),
  "sms_from": "1234567890"
}
```

You can also define (globally) `sms_from` which is the phone number or name that your SMS will be sent from. You can generate a phone number for your application in the Vonage dashboard.

You should define a `to_vonage` method on the notification class to specify how to build the sms notification content.

```python
from masonite.notification.components import VonageComponent

class Welcome(Notification):

    def to_vonage(self, notifiable):
        return VonageComponent().text("Welcome!")

    def via(self, notifiable):
        return ["vonage"]
```

#### Options

If the SMS notification contains unicode characters, you should call the unicode method when constructing the notification

```python
VonageComponent().text("Welcome unicode message!").set_unicode()
```

The global `sms_from` number can be overriden inside the notification class:

```python
VonageComponent().text("Welcome!").sms_from("+123 456 789")
```

#### Routing to notifiable

You should define the related `route_notification_for_vonage` method on your notifiable to return a phone number or a list of phone numbers to send the notification to.

```python
class User(Model, Notifiable):

    def route_notification_for_vonage(self, notification):
        return self.phone
        # or return [self.mobile_phone, self.land_phone]
```

#### Routing to anonymous

To send a SMS notification without having a notifiable entity you must use the `route` method

```python
notification.route("vonage", "+33612345678").notify(Welcome())
```

## Saving notifications

Notifications can be stored in your application database when sent to `Notifiable` entities. The notification is stored in a `notifications` table. This table will contain information such as the notification type as well as a JSON data structure that describes the notification.

To store a notification in the database you should define a `to_database` method on the notification class to specify how to build the notification content that will be persisted.

```python
class Welcome(Notification):

    def to_database(self, notifiable):
        return {"data": "Welcome {0}!".format(notifiable.name)}

    def via(self):
        return ["mail", "database"]
```

This method should return `str`, `dict` or `JSON` data (as it will be saved into a `TEXT` column in the notifications table). You also need to add `database` channel to the `via` method to enable database notification storage.

### Initial setup

Before you can store notifications in database you must create the database `notifications` table.

```
$ python craft notification:table
```

Then you can migrate your database

```
$ python craft migrate
```

The ORM Model describing a Notification is `DatabaseNotification` and has the following fields:

* `id` is the primary key of the model (defined with UUID4)
* `type` will store the notification type as a string (e.g. `WelcomeNotification`)
* `read_at` is the timestamp indicating when notification has been read
* `data` is the serialized representation of `to_database()`
* `notifiable` is the relationship returning the `Notifiable` entity a notification belongs to (e.g. `User`)
* `created_at`, `updated_at` timestamps

### Querying notifications

A notifiable entity has a `notifications` relationship that will returns the notifications for the entity:

```python
user = User.find(1)
user.notifications.all() # == Collection of DatabaseNotification belonging to users
```

You can directly get the unread/read notifications:

```python
user = User.find(1)
user.unread_notifications.all() # == Collection of user unread DatabaseNotification
user.read_notifications.all() # == Collection of user read DatabaseNotification
```

### Managing notifications

You can mark a notification as read or unread with the following `mark_as_read` and `mark_as_unread` methods

```python
user = User.find(1)

for notification in user.unread_notifications.all():
    notification.mark_as_read()
```

{% hint style="info" %}
Finally, keep in mind that database notifications can be used as any `Masonite ORM` models, meaning you can for example make more complex queries to fetch notifications, directly on the model.
{% endhint %}

```python
from masonite.notification import DatabaseNotification

DatabaseNotification.all()
DatabaseNotification.where("type", "WelcomeNotification")
```

## Broadcasting Notifications

If you would like to broadcast the notification then you need to:

* inherit the `CanBroadcast` class and specify the `broadcast_on` method
* define a `to_broadcast` method on the notification class to specify how to build the notification content that will be broadcasted

```python
class Welcome(Notification, CanBroadcast):

    def to_broadcast(self, notifiable):
        return f"Welcome {notifiable.name} !"

    def broadcast_on(self):
        return "channel1"

    def via(self, notifiable):
        return ["broadcast"]
```

### Broadcasting to notifiables

By default notifications will be broadcasted to channel(s) defined in `broadcast_on` method but you can override this per notifiable by implementing `route_notification_for_broadcast` method on your notifiable:

```python
class User(Model, Notifiable):

    def route_notification_for_broadcast(self, notification):
        return ["general", f"user_{self.id}"]
```

### Broadcasting to anonymous

```python
notification.route("broadcast", "channel1").notify(Welcome())
```

## Adding a new driver

Masonite ships with a handful of notification channels, but you may want to write your own drivers to deliver notifications via other channels. Masonite makes this process simple.

### Creating the driver

Two methods need to be implemented in order to create a notification driver: `send` and `queue`.

```python
from masonite.notification.drivers import BaseDriver

class VoiceDriver(BaseDriver):

    def send(self, notifiable, notification):
        """Specify here how to send the notification with this driver."""
        data = self.get_data("voice", notifiable, notification)
        # do something
```

`get_data()` method will be available and will return the data defined in the `to_voice()` method of the notification.

### Registering the driver

As any drivers it should be registered, through a custom provider for example:

```python
from masonite.providers import Provider

class VoiceNotificationProvider(Provider):

    def register(self):
        self.application.make("notification").add_driver("voice", VoiceDriver(self.application))
```

Then you could scaffold this code into a new Masonite package so that community can use it 😉 !

## Advanced Usage

### Dry run

You can enable dry notifications to avoid notifications to be sent. It can be useful in some cases (background task, production commands, development, testing...)

```python
user.notify(WelcomeNotification(), dry=True)
# or
notification.send(WelcomeNotification(), dry=True)
```

### Ignoring errors

When `fail_silently` parameter is enabled, notifications sending will not raise exceptions if an error occurs.

```python
user.notify(WelcomeNotification(), fail_silently=True)
# or
notification.send(WelcomeNotification(), fail_silently=True)
```

### Overriding channels

Channels defined in `via()` method of the notification can be overriden at send:

```python
class Welcome(Notification):
    def to_mail(self, notifiable):
        #...

    def to_slack(self, notifiable):
        #...

    def to_database(self, notifiable):
        #...

    def via(self):
        """Default behaviour is to send only by email."""
        return ["mail"]
```

Using `channels` parameter we can send to other channels (if correctly defined in notification class):

```python
user.notify(Welcome(), channels=["slack", "database"])
# or
notification.send(Welcome(), channels=["slack", "database"])
```
