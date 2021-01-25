# Masonite Notifications

## Introduction

Masonite Notifications help you to easily add sending notifications to your project. These notifications could be sending an email or a Slack message. This package is designed to be extremely simple and modular. New notification drivers can be added through third party integrations (Masonite packages).

### Features
- Send `E-mail`, `Slack` and `SMS` notifications
- Store notifications in a database so they may be displayed in your web interface.
- Queue notifications
- Broadcast notifications

## Installation

In order to get started using Masonite Notifications we need to install the package:

```text
$ pip install masonite-notifications
```

And then add the provider to our `PROVIDERS` list (after `ORMProvider`):

```python
from masonite.notifications import NotificationProvider
...

PROVIDERS = [
    # ...
    NotificationProvider,
    # ...
]
```
Finally you must publish the configuration file:
```text
$ python craft publish NotificationProvider --tag=config
```

Thats it! Let's see how it works! There are a few concepts we'll need to cover so you can fully understand how Notifications work. First we will cover the high level stuff and then slowly work down into the lower level implementations.

## Creating a Notification

In order to send a notification, we need to create a notification class. We can do this simply with a new craft command.

```text
$ python craft notification Welcome
```

This will create a `WelcomeNotification` file in the `app/notifications` directory. Feel free to move it wherever you like though.

This will create a class like so:

```python
''' A WelcomeNotification Notification '''
from masonite.notifications import NotificationFacade

class WelcomeNotification(NotificationFacade):

    def to_mail(self, notifiable):
        pass

    def to_database(self, notifiable):
        pass

    def via(self, notifiable):
        """Defines the notification's delivery channels."""
        return ["mail"]
```

## Defining Notifiables entities

ORM Models can be defined as `Notifiable` to allow notifications to be sent to them. The most common use case is to set `User` model as `Notifiable` as we often send notifications to user.

### Set Model as Notifiable

To set a Model as Notifiable, just add the `Notifiable` mixin to it:

```python
from masonite.notifications import Notifiable

class User(Model, Notifiable):
    # ...
```

You can now send notifications to it with `user.notify()` method.

### Define routing

Then you can define how notifications should be routed for the different channels. This is always done
by defining a `route_notification_for_{driver}` method. This method should returns the recipient field data of
the notification or a list of recipient fields data.

For example, with `mail` driver you can define:

```python
from masonite.notifications import Notifiable

class User(Model, Notifiable):
    ...
    def route_notification_for_mail(self):
        return self.email
```

This is actually the default behaviour of the mail driver so you won't have to write that but you can customize it
to your needs if your User model don't have `email` field or if you want to add some logic to get the recipient email.

## Sending notifications

### Configure delivery channels

Every notification class has a `via` method that determines on which channels the notification will be delivered. Notifications may be sent on the `mail`, `database`, `broadcast`, `slack` and `vonage` channels.

{% hint style="info" %}
If you would like to use an other delivery channel, feel free to check if a community driver has been developed for it or [create your own driver and share it with the community](#) !
{% endhint %}

`via` method should returns a list of the channels you want your notification to be delivered on. This method receives a `notifiable` instance.

```python
from masonite.notifications import NotificationFacade

class WelcomeNotification(NotificationFacade):
    ...
    def via(self, notifiable):
        return ["mail", "database"]
        # or to handle more use cases
        # return ["mail"] if notifiable.role == "admin" else ["mail", "database"]
```

When sending the notification it will be automatically sent to each channel.

### Sending to Notifiables

There is two ways of sending a Notification:

- through `notify` method of a Notifiable entity

```python
# RegisterController.py
def register(self, view: View):
    user = self.request.user()
    user.notify(WelcomeNotification())
    return view.render("confirmation")
```

- through `send` method of the Notification service

```python
# RegisterController.py
from masonite.notifications import Notification

def register(self, view: View, notification: Notification):
    user = self.request.user()
    notification.send(user, WelcomeNotification())
    return view.render("confirmation")
```

This last method comes handy when sending notifications to a batch of entities (which is not directly possible with previous without for loop):

```python
users = User.all()
Notification.send(users, WelcomeNotification())
```

### Sending to Anonymous Users / On-demand

Sometimes you want to send a notification to someone not registered as a User in your database, or which is not related to a `Notifiable` entity. It is possible with the `Notification` service:

```python
# RegisterController.py
from masonite.notifications import Notification

def register(self, view: View, notification: Notification):
    notification.route('mail', 'sam@masonite.com').send(WelcomeNotification())
    return view.render("confirmation")
```

If the notification needs to be delivered to multiple channels you can chain the different routes:

```python
# RegisterController.py
from masonite.notifications import Notification

def register(self, view: View, notification: Notification):
    notification.route('mail', 'sam@masonite.com') \
        .route('slack', '#general') \
        .send(WelcomeNotification())
    return view.render("confirmation")
```

{% hint style="warning" %}
`database` channel cannot be used with those notifications because no Notifiable
entity is attached to it.
{% endhint %}

## Queueing notifications

If you would like to queue the notification then you just need to inherit the `ShouldQueue` class and it will automatically send your notifications into the queue to be processed later. This is a great way to speed up your application:

```python
from masonite.notifications import NotificationFacade
from masonite.queues import ShouldQueue

class WelcomeNotification(NotificationFacade, ShouldQueue):
    # ...
```

## Mail Notifications

If a notification supports being sent as an email, you should define a `to_mail` method on the notification class to specify how to build the notification content.
This method should returns either:

- a [Mailable](/useful-features/mail#mailable-classes)
- a MailComponent based email

### Using Mailable

```python
from masonite.notifications import NotificationFacade

# here WelcomeEmail is a Mailable
class WelcomeNotification(NotificationFacade):
    def to_mail(self, notifiable):
        return WelcomeEmail(notifiable)
```
then in the WelcomeEmail Mailable class you can use notifiable data.

### Using Mail Components

You can:
- combine existing handy components to quickly create a cool email
- use a view to render your email.

```python
from masonite.notifications import NotificationFacade
from masonite.notifications.components import MailComponent

class WelcomeNotification(NotificationFacade):

    def to_mail(self, notifiable):
        return MailComponent()
            .subject('New email subject') \
            .panel('GBALeague.com') \
            .heading(f' {notifiable.name} you have created a new account!') \
            .line('We greatly value your service!') \
            .line('Attached is an invoice for your recent purchase') \
            .action('Sign Back In', href="http://gbaleague.com") \
            .line('See you soon! Game on!') \
```
This will build the email with the different blocks in the specified order.

```python
from masonite.notifications import NotificationFacade
from masonite.notifications.components import MailComponent

class WelcomeNotification(NotificationFacade):

    def to_mail(self, notifiable):
        return MailComponent()
            .subject('New email subject') \
            .view('emails.welcome', {"name": notifiable.name}) \
```
This will render the mail with the view located at `resources/templates/emails/welcome.html`.

You can find below the different components that you can use when building an email based on `MailComponent`

| Method       | Description                                                                                                                                                                    | Example                                                                             |
| :----------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| .line\(\)    | Creates a single line of text like text you would see in a paragraph tag                                                                                                       | line\('this is a line of text'\)                                                    |
| .action\(\)  | This creates a clickable looking button. The kwargs include `href` and `style`. The styles are bootstraps button styles to include `default`, `success`, `danger`, `info` etc. | action\('Click Me', href="[http://google.com](http://google.com)", style="danger"\) |
| .view\(\)    | This is the normal view object here so you can pass in any templates and dictionary you need.                                                                                  | .view\('mail/header', {'key': 'value'}\)                                            |
| .panel\(\)   | This creates a grey background header panel.                                                                                                                                   | .panel\('Some Header'\)                                                             |
| .heading\(\) | Creates a header                                                                                                                                                               | .heading\('Welcome!'\)                                                              |
| .subject\(\) | The subject of the email                                                                                                                                                       | .subject\('New Account!'\)                                                          |
| .send_from\(\) | The sender email of the email. A name can also be specified.                                                                                                                                                      | .send_from\('sam@masonite.com',name="Sam"\)                                                          |
| .reply_to\(\) | The reply-to field of the email. An email or a list of emails can be specified.                                                                                                                                                      | .reply_to\(['sam@masonite.com', 'joe@masonite.com']\)                                                          |
| .driver\(\)  | Override the driver used to send the email. By default, the email notification will be sent using the default driver defined in the configuration                                                                                                                                   | .driver\('mailgun'\)                                                                |

## Database Notifications

Notifications can be stored in your application database when sent to `Notifiable` entities. The notification is stored
in a `notifications` table. This table will contain information such as the notification type as well as a JSON data structure that describes the notification. The ORM Model describing a Notification is `DatabaseNotification`.

### Creating notifications table
Before you can store notifications in database you must create the database `notifications` table.
A handy command is available to publish the table migration file.
```text
$ python craft publish NotificationProvider --tag migrations
```
Then you can migrate your database
```
$ python craft migrate
```

### Formatting database notifications
To store a notification in the database you should define a `to_database` method on the notification class to specify how to build the notification content that will be persisted.
```python
class WelcomeNotification(NotificationFacade):
    # ...
    def to_database(self, notifiable):
        return {"data": "Welcome {0}!".format(notifiable.name)}

    def via(self):
        return ["mail", "database"]
```
This method should return `str`, `dict` or `JSON` data (as it will be saved into a `TEXT` column in the notifications table).
You also need to add `database` channel to the `via` method to enable database notification storage.

### Notification Model
The notification model has the following fields:
- `id` is the primary key of the model (defined with UUID4)
- `type` will store the notification type as a string (e.g. `WelcomeNotification`)
- `read_at` is the timestamp indicating when notification has been read
- `data` is the serialized representation of `to_database()`
- `notifiable` is the relationship returning the `Notifiable` entity a notification belongs to (e.g. `User`)
- `created_at`, `updated_at` timestamps

### Fetching notifications
A notifiable entity has a `notifications` relationship that will returns the notifications for the entity:
```python
user = User.find(1)
user.notifications.all() # == Collection of DatabaseNotification belonging to users
```

{% hint style="info" %}
By default, notifications will be sorted by the `created_at` timestamp with the most recent notifications at the beginning of the collection.
{% endhint %}

If you want to get only the unread notifications or the read notifications you can use the two following helpers on a
notifiable entity:
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
Finally, keep in mind that database notifications can be used as any `Masonite ORM` models, meaning you
can for example make more complex queries to fetch notifications, directly on the model.
{% endhint %}
```python
from masonite.notifications import DatabaseNotification

DatabaseNotification.all()
DatabaseNotification.where("type", "WelcomeNotification")
```

## Broadcast Notifications
TODO

## Slack Notifications

Notifications can be sent to your Slack workspace very easily. This can be achieved via [different ways](https://api.slack.com/messaging/sending#sending_methods). Here in Masonite, two options are available:
- Slack Incoming Webhooks [more here](https://api.slack.com/messaging/webhooks)
- Slack Web API [more here](https://api.slack.com/methods/chat.postMessage)

### Configure Slack integration

#### Slack Incoming Webhooks
You will need to [configure an "Incoming Webhook"](https://masoniteproject.slack.com/apps/A0F7XDUAZ-incoming-webhooks) integration for your Slack workspace. This integration will provide you with a URL you may use when routing Slack notifications. This URL will target a specific Slack channel.

#### Slack Web API
You will need to [generate a token](https://api.slack.com/web#slack-web-api__authentication) to interact with your Slack workspace.
{% hint style="info" %}
This token should have at minimum the `channels:read`, `chat:write:bot`, `chat:write:user` and `files:write:user` permission scopes. If your token does not have these scopes then parts of this feature will not work.
{% endhint %}

Then you can define this token globally in `config/notifications.py` file as `SLACK_TOKEN` environment variable. Or you can configure different tokens (with eventually different scopes) per notifications (more here)[TODO].

### Formatting Notifications

If a notification supports being sent to Slack, you should define a `to_slack` method on the notification class to specify how to build the notification content.

```python
from masonite.notifications import NotificationFacade
from masonite.notifications.components import SlackComponent

class WelcomeNotification(NotificationFacade):
    def to_slack(self, notifiable):
        return SlackComponent().text('Masonite Notification: Read The Docs!, https://docs.masoniteproject.com/') \
            .channel('#bot') \
            .as_user('Masonite Bot') \

    def via(self):
        return ["email", "slack"]
```
You can see that the notification content is based on the `SlackComponent` class. All the options available are listed here:

| Method                | Description                                                                                                                                                                                                                                                                   | Example                                                                                                |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| .text\(\)             | The text you want to show in the message                                                                                                                                                                                                                                      | .text\('Welcome to Masonite!'\)                                                                        |
| .to\(\)          | The channel you want to broadcast to. If the value you supply starts with a \# sign then Notifications will make a POST request with your token to the Slack channel list API and get the channel ID. You can specify the channel ID directly if you don't want this behavior | .to\('\#general'\) .to\('CHSUU862'\)                                                         |
| .send_from\(\)          | The username you want to show as the message sender. You can also specify either the `url` or `icon` that will be displayed as the sender.                                                                                                                                                                                                                             | .send_from\('Masonite Bot', icon=":ghost:"\)                                                                             |
| .as_current_user\(\)  | This sets a boolean value to True on whether the message should show as the currently authenticated user.                                                                                                                                                                     | .as_current_user\(\)                                                                                   |
| .link_names\(\)  | This enables finding and linking channel names and usernames in message.                                                                                                                                                                      | .link_names\(\)                                                                                   |
| .can_reply\(\)  | This auhtorizes replying back to the message.                                                                                                                                                                      | .can_reply\(\)                                                                                   |
| .without_markdown\(\) | This will not parse any markdown in the message. This is a boolean value and requires no parameters.                                                                                                                                                                          | .without_markdown\(\)                                                                                  |
| .unfurl_links\(\)      | This enable showing message attachments and links previews. Usually slack will show an icon of the website when posting a link. This enables that feature for the current message.                                                                      | .unfurl_links\(\)                                                                                       |
| .as_snippet\(\)       | Used to post the current message as a snippet instead of as a normal message. This option has 3 keyword arguments. The `file_type`, `name`, and `title`. This uses a different API endpoint so some previous methods may not be used.                                         | .as_snippet\(file_type='python', name='snippet', title='Awesome Snippet'\)                             |
| .token\(\)            | Override the globally configured token.                                                                                                                                                                                                              | .token\('xoxp-359926262626-35...'\)                                                                    |


### Advanced Formatting
Slack notifications can use [Slack Blocks Kit](https://api.slack.com/block-kit/building) to build more complex notifications.
Before using this you just have to install `slackblocks` python API to handle Block Kit formatting.
```text
$ pip install slackblocks
```

Then you can import most of the blocks available in Slack documentation and start building your notification. You need to use the `block()` option. Once again you can chain as many blocks as you want.
```python
from masonite.notifications import NotificationFacade
from masonite.notifications.components import SlackComponent
from slackblocks import HeaderBlock, ImageBlock, DividerBlock


class WelcomeNotification(NotificationFacade):
    def to_slack(self, notifiable):
        return SlackComponent() \
            .text('Notification text') \
            .channel('#bot') \
            .block(HeaderBlock("Header title")) \
            .block(DividerBlock()) \
            .block(ImageBlock("https://path/to/image", "Alt image text", "Image title"))

    def via(self):
        return ["email", "slack"]
```

You can find all blocks name and options in [`slackblocks` documentation](https://github.com/nicklambourne/slackblocks#usage) and more information in [Slack blocks list](https://api.slack.com/reference/block-kit/blocks).

{% hint style="warning" %}
Some blocks or elements might not be yet available in `slackblocks`, but most of them should be there.
{% endhint %}


### Routing your notifications

You should define the related `route_notification_for_slack` method on your notifiable to return either
- a webhook URL or a list of webhook URLs (if you're using [Incoming Webhooks](/#slack-incoming-webhooks))
- a channel name/ID or a list of channels names/IDs (if you're using [Slack Web API](/#slack-web-api))

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

To send a Slack notification without having a notifiable entity you must use the `route` method
```python
notification.route("slack", "#general").notify(
    WelcomeNotification()
)
```
The second parameter can be a channel name, a channel ID or a webhook URL.

{% hint style="warning" %}
When specifying channel names you must keep `#` in the name as in the example. Based on this name
a reverse lookup will be made to find the corresponding Slack channel ID. If you want to avoid this extra
call, you can specify directly the channel ID (right click on Slack channel > Copy Name > the ID is at the end of url)
{% endhint %}


## SMS Notifications

Sending SMS notifications in Masonite is powered by [Vonage](https://www.vonage.com/communications-apis/sms/) (formerly Nexmo).
Before you can send notifications via Vonage, you need to install the `vonage` Python client.
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

### Formatting SMS Notifications
If a notification supports being sent as a SMS, you should define a `to_vonage` method on the notification class to specify how to build the notification content.

```python
from masonite.notifications import NotificationFacade
from masonite.notifications.components import VonageComponent

class WelcomeNotification(NotificationFacade):
    def to_vonage(self, notifiable):
        return VonageComponent().text("Welcome!")
```

If the SMS notification contains unicode characters, you should call the unicode method when constructing the notification
```python
# WelcomeNotification.py
    def to_vonage(self, notifiable):
        return VonageComponent().text("Welcome unicode message!").set_unicode()
```

### Overriding "from" setting
The global `sms_from` number can be overriden in the notification:

```python
# WelcomeNotification.py
    def to_vonage(self, notifiable):
        return VonageComponent().text("Welcome!").sms_from("+123 456 789")
```

### Routing your notifications

You should define the related `route_notification_for_vonage` method on your notifiable to return a phone number or a list of phone numbers to send the notification to.

```python
class User(Model, Notifiable):

    def route_notification_for_vonage(self, notification):
        return self.phone
        # or return [self.mobile_phone, self.land_phone]
```

To send a SMS notification without having a notifiable entity you must use the `route` method
```python
notification.route("vonage", "+33612345678").notify(
    WelcomeNotification()
)
```

## Custom notifications drivers

Masonite ships with a handful of notification channels, but you may want to write your own drivers to deliver notifications via other channels. Masonite makes this process simple.

### Create a Notification driver class

Two methods need to be implemented in order to create a notification driver: `send` and `queue`.

```python
from masonite.drivers import BaseDriver
from masonite.notifications import NotificationContract

class NotificationVoiceDriver(BaseDriver, NotificationContract):

    def send(self, notifiable, notification):
        """Specify here how to send the notification with this driver."""
        data = self.get_data("voice", notifiable, notification)

    def queue(self, notifiable, notification):
        """Specify here how to queue the notification with this driver."""
        data = self.get_data("voice", notifiable, notification)

```
`get_data()` method will be available and will return the data defined in the `to_voice()` method of the notification.


### Register the driver into your application.

As any drivers it should be registered, through a custom provider for example:

```python
from masonite.provider import ServiceProvider

class VoiceNotificationProvider(ServiceProvider):
    wsgi = False

    def register(self):
        self.app.bind("NotificationVoiceDriver", NotificationVoiceDriver)
```
This provider should be added after (?) the `NotificationProvider`. That's it !

{% hint style="info" %}
Feel free to browse [notifications drivers code](https://github.com/MasoniteFramework/notifications/tree/master/) to better understand how to develop your own driver.
{% endhint %}

Then you could scaffold this code into a new [Masonite package](/advanced/creating-packages) so
that community can use it 😉 !

## Advanced Usage

### Dry run
You can enable sending dry notifications to avoid notifications to be send (only logged) in two places:
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

### Overriding notifications channels
Even if channels that a notification will be sent to is defined initially in `via()` method of the notification,
you can override this behaviour when sending:

```python

class WelcomeNotification(NotificationFacade):
    # ...
    def via(self):
        return ["mail"]

user.notify(WelcomeNotification(), channels=["slack", "database"])
# or
notification.send(WelcomeNotification(), channels=["slack", "database"])
```
This only makes sense for notifiables and not for on-demand notifications.
