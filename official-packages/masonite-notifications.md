# Masonite Notifications

## Introduction

Masonite Notifications can easily add new notification sending semantics for your Masonite project. These notifications could be sending an email or a slack message. This package is designed to be extremely simple and modular. New notification drivers can be added through third party integrations (Masonite packages).

Masonite provides support for sending notifications across mail and Slack. Notifications may also be stored in a database so they may be displayed in your web interface.

## Installation

In order to get started using Masonite Notifications, we first have to pip install it:

```text
$ pip install masonite-notifications
```

And then add the provider to our `PROVIDERS` list:

```python
from notifications.providers import NotificationProvider
...

PROVIDERS = [
    ...
    NotificationProvider,
    ...
]
```

Thats it! Let's see how it works!

## Usage

There are a few concepts we'll need to cover so you can fully understand how Notifications work. First we will cover the high level stuff and then slowly work down into the lower level implementations. For the purposes of this documentation, we will walk through how to setup a welcome notification so we can send an email when a user signs up.

## Creating a Notification

In order to use it we can create a notification class. We can do this simply with a new craft command.

```text
$ craft notification WelcomeNotification
```

This will create a notification in the `app/notifications` directory. Feel free to move it wherever you like though.

This will create a class like so:

```python
''' A WelcomeNotification Notification '''
from masonite.notifications import Notification

class WelcomeNotification(Notification):

    def to_mail(self, notifiable):
        pass

    def to_database(self, notifiable):
        pass

    def via(self, notifiable):
        """Defines the notification's delivery channels."""
        return ["mail"]
```

## Defining Notifiables entities

Models can be defined as `Notifiable` to allow notifications to be sent to them. The most common
use case would be to set `User` model as `Notifiable`.

### Set Model as Notifiable

To set a Model as Notifiable, just add the `Notifiable` mixin to it:

```python
from masonite.notifications import Notifiable

class User(Model, Notifiable):
    ...
```

You can now send notifications to it with `user.notify()` method. When using `database` notification
driver some getters are available to fetch user notifications (read/unread).

### Define routing

Then you can define how notifications should be routed for the different channels. This is always done
by defining a `route_notification_for_{driver}` method. This method should returns the recipient field data of
the notification or a list of recipient fields data.

For example, with `mail` driver you can define:

```python
class User(Model, Notifiable):
    ...
    def route_notification_for_mail(self):
        return self.email
```

This is actually the default behaviour of the mail driver so you won't have to write that but you can customize it
to your needs if you don't have the same field name or if you want to add some logic to get the recipient email.

## Sending a Notification

### Configure delivery channels

Every notification class has a `via` method that determines on which channels the notification will be delivered. Notifications may be sent on the `mail`, `database`, `broadcast`, and `slack` channels.

{% hint style="info" %}
If you would like to use an other delivery channel, feel free to check if a community driver has been developed for or [create your own driver and share it with the community](#) !
{% endhint %}

`via` method should returns a list of the channels you want your notification to be delivered on.
This method receives a `notifiable` instance.

```python
class WelcomeNotification(Notification):
    ...
    def via(self, notifiable):
        return ["mail", "database"]
        # or to handle more use cases
        # return ["mail"] if notifiable.role == "admin" else ["mail", "database"]
```

When sending the notification it will be automatically sent to each channel.

### To Notifiables

There is two ways of sending a Notification:

- through `notify` method of a Notifiable entity

```python
# RegisterController.py
def register(self, view: View):
    user = self.request.user()
    user.notify(WelcomeNotification())
    return view.render("confirmation")
```

- through `send` method of the Notification module

```python
# RegisterController.py
def register(self, view: View, notification: Notification):
    user = self.request.user()
    Notification.send(user, WelcomeNotification())
    return view.render("confirmation")
```

This last method comes handy when sending notifications to a batch of entities (which is not directly
possible with previous without for loop):

```python
users = User.all()
Notification.send(users, WelcomeNotification())
```

### To Anonymous Users / On-demand

Sometimes you want to send a notification to someone not registered as a User in your database,
or which is not related to a Notifiable entity. It is possible with the `Notification` module:

```python
# RegisterController.py
def register(self, view: View, notification: Notification):
    Notification.route('mail', 'sam@masonite.com').send(WelcomeNotification())
    return view.render("confirmation")
```

If the notification is delivered to multiple channels you can define the different routes
at the same time:

```python
# RegisterController.py
def register(self, view: View, notification: Notification):
    Notification.route('mail', 'sam@masonite.com') \
        .route('slack', '#general') \
        .send(WelcomeNotification())
    return view.render("confirmation")
```

{% hint style="warning" %}
`database` channel cannot be used with those notifications because no Notifiable
entity is attached to it.
{% endhint %}

### Queue notifications

**To (re)implement and document**

If you would like to queue the notification then you just need to inherit the `ShouldQueue` class and it will automatically send your notifications into the queue to be processed later. This is a great way to speed up your application:

```python
from masonite.notifications import Notification
from masonite.queues import ShouldQueue

class WelcomeNotification(Notifiable, ShouldQueue):
    ...
```

## Mail Notifications

If a notification supports being sent as an email, you should define a `to_mail` method on the notification class.
This method should returns either:

- a [Mailable](#)
- a MailComponent combination
- (soon) Markdown
- (soon) MJML support (maybe done through view actually)

### Using Mailable

```python
# WelcomeEmail is a Mailable
def to_mail(self, notifiable):
    return WelcomeEmail(notifiable.name).to(notifiable.email)
```

### Using Mail Components

You can combine existing handy components to quickly create a cool email or you can also use a view
to render your email.

```python
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

```python
# will render the mail with resources/templates/emails/welcome.html
def to_mail(self, notifiable):
    return MailComponent()
        .subject('New email subject') \
        .view('emails.welcome', {"name": notifiable.name}) \
```

The email will looked like:
**ADD IMAGE**

Let's walk through the different options to build an email notification and what they do.

| Method       | Description                                                                                                                                                                    | Example                                                                             |
| :----------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| .line\(\)    | Creates a single line of text like text you would see in a paragraph tag                                                                                                       | line\('this is a line of text'\)                                                    |
| .action\(\)  | This creates a clickable looking button. The kwargs include `href` and `style`. The styles are bootstraps button styles to include `default`, `success`, `danger`, `info` etc. | action\('Click Me', href="[http://google.com](http://google.com)", style="danger"\) |
| .view\(\)    | This is the normal view object here so you can pass in any templates and dictionary you need.                                                                                  | .view\('mail/header', {'key': 'value'}\)                                            |
| .panel\(\)   | This creates a grey background header panel.                                                                                                                                   | .panel\('Some Header'\)                                                             |
| .heading\(\) | Creates a header                                                                                                                                                               | .heading\('Welcome!'\)                                                              |
| .subject\(\) | The subject of the email                                                                                                                                                       | .subject\('New Account!'\)                                                          |
| .driver\(\)  | The driver you want to use to send the email                                                                                                                                   | .driver\('mailgun'\)                                                                |

## Database Notifications

TODO

## Broadcast Notifications

TODO

## Slack Notifications

Out of the box, Masonite notifications comes with Slack support as well in case we want to send a message to a specific slack group.

{% hint style="info" %}
NOTE: In order to use this feature fully, you'll need to generate a token from Slack. This token should have at minimum the `channels:read`, `chat:write:bot`, `chat:write:user` and `files:write:user` permission scopes. If your token does not have these scopes then parts of this feature will not work.
{% endhint %}

Going back to our WelcomeNotification, we can simply specify a new method called `slack`.

```python
class WelcomeNotification(Notifiable):

    def mail(self):
        return self.subject('New account signup!') \
            .driver('smtp') \
            .panel('GBALeague.com') \
            .heading('You have created a new account!') \
            .line('We greatly value your service!') \
            .line('Attached is an invoice for your recent purchase') \
            .action('Sign Back In', href="http://gbaleague.com") \
            .line('See you soon! Game on!') \
            .view('/notifications/snippets/mail/heading',
                  {'message': 'Welcome To The GBA!'})

      def slack(self):
          pass
```

Notice the new slack method at the bottom. we will use this method to build our slack notification. Again we will show you a full example and then run through all the methods:

```python
class WelcomeNotification(Notifiable):

    def mail(self):
        return self.subject('New account signup!') \
            .driver('smtp') \
            .panel('GBALeague.com') \
            .heading('You have created a new account!') \
            .line('We greatly value your service!') \
            .line('Attached is an invoice for your recent purchase') \
            .action('Sign Back In', href="http://gbaleague.com") \
            .line('See you soon! Game on!') \
            .view('/notifications/snippets/mail/heading',
                  {'message': 'Welcome To The GBA!'})

    def slack(self):
        return self.token(os.getenv('BOT')) \
            .text('Masonite Notification: Read The Docs!, https://docs.masoniteproject.com/') \
            .channel('#bot') \
            .as_user('Masonite Bot') \
            .icon(':fire:') \
            .button('Sure!', "https://docs.masoniteproject.com/")
```

### Options

| Method                | Description                                                                                                                                                                                                                                                                   | Example                                                                                                |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| .token\(\)            | This is your Slack token that has the correct permission scopes.                                                                                                                                                                                                              | .token\('xoxp-359926262626-35...'\)                                                                    |
| .text\(\)             | The text you want to show in the message                                                                                                                                                                                                                                      | .text\('Welcome to Masonite!'\)                                                                        |
| .channel\(\)          | The channel you want to broadcast to. If the value you supply starts with a \# sign then Notifications will make a POST request with your token to the Slack channel list API and get the channel ID. You can specify the channel ID directly if you don't want this behavior | .channel\('\#general'\) .channel\('CHSUU862'\)                                                         |
| .as_user\(\)          | The username you want to show as the message                                                                                                                                                                                                                                  | .as_user\('Masonite Bot'\)                                                                             |
| .icon\(\)             | The icon you want to show. This needs to be a Slack emoji                                                                                                                                                                                                                     | .icon\(':fire:'\)                                                                                      |
| .as_current_user\(\)  | This sets a boolean value to True on whether the message should show as the currently authenticated user.                                                                                                                                                                     | .as_current_user\(\)                                                                                   |
| .without_markdown\(\) | This will not parse any markdown in the message. This is a boolean value and requires no parameters.                                                                                                                                                                          | .without_markdown\(\)                                                                                  |
| .dont_unfurl\(\)      | This sets a boolean to False on whether the message should show any attachments. Usually slack will show an icon of the website when posting a link. This disables that feature for the current message.                                                                      | .dont_unfurl\(\)                                                                                       |
| .as_snippet\(\)       | Used to post the current message as a snippet instead of as a normal message. This option has 3 keyword arguments. The `file_type`, `name`, and `title`. This uses a different API endpoint so some previous methods may not be used.                                         | .as_snippet\(file_type='python', name='snippet', title='Awesome Snippet'\)                             |
| .comment\(\)          | Only used when using the .as_snippet\(\) method. This will set a comment on the snippet.                                                                                                                                                                                      | .comment\('Great Snippet'\)                                                                            |
| .button\(\)           | Used to create action buttons under a message. This requires `text` and a `url` but can also contain `style`, and `confirm`                                                                                                                                                   | .button\('Sure!', '[http://google.com](http://google.com)', style='primary', confirm='Are you sure?'\) |
| .dry\(\)              | Sets all the necessary fields but does not actually send the email. This is great for testing purposes. This takes no parameters                                                                                                                                              | .dry\(\)                                                                                               |

## Adding Notifications drivers

**Explain here how to develop new notifications drivers in external packages**

Masonite ships with a handful of notification channels, but you may want to write your own drivers to deliver notifications via other channels. Masonite makes this process simple.

### Create a Notification driver class

**TODO**

```python
from masonite.drivers import BaseDriver
from masonite.notifications import NotificationContract

class VoiceNotification(BaseDriver, NotificationContract)

    def send(self, notifiables, notification):
        pass

```

- template
- send(notifiables, notification)
- get_data()

### Register it

As any drivers you should register it into the Notification service provider ?
Finally the best would be to scaffold this into a new Masonite package (**show resources**) so that community can use it.
