# Masonite Notifications

## Introduction

Masonite Notifications can easily add new notification sending semantics for your Masonite project. These notifications could be sending an email or a slack message. This package is designed to be extremely simple and modular so allow new notification abilities to be added to it through third party integrations.

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
from notifications import Notifiable

class WelcomeNotification(Notifiable):

    def mail(self):
        pass
```

## Building Our Mail Notification

Let's now walk through how to build a notification so we can send the email to our user.

Since our notification inherits from `Notifiable`, we have access to a few methods we will use to build the notification. We'll show a final product of what it looks like since it's pretty straight forward but we'll walk through it after:

```python
from notifications import Notifiable
import os


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
```

Notice here we are calling a few methods such as `driver`, `panel`, `line`, etc. If we send this message it will look like:

![](../.gitbook/assets/screen-shot-2018-07-28-at-9.46.23-pm.png)

Not bad. We can use this logic to easily build up emails into a nice format simply.

### Options

Let's walk through the different options to build an email notification and what they do.

| Method | Description | Example |
| :--- | :--- | :--- |
| .line\(\) | Creates a single line of text like text you would see in a paragraph tag | line\('this is a line of text'\) |
| .action\(\) | This creates a clickable looking button. The kwargs include `href` and `style`. The styles are bootstraps button styles to include `default`, `success`, `danger`, `info` etc. | action\('Click Me', href="[http://google.com](http://google.com)", style="danger"\) |
| .view\(\) | This is the normal view object here so you can pass in any templates and dictionary you need. | .view\('mail/header', {'key': 'value'}\) |
| .panel\(\) | This creates a grey background header panel. | .panel\('Some Header'\) |
| .heading\(\) | Creates a header | .heading\('Welcome!'\) |
| .subject\(\) | The subject of the email | .subject\('New Account!'\) |
| .dry\(\) | Sets all the necessary fields but does not actually send the email. This is great for testing purposes. This takes no parameters | .dry\(\) |
| .driver\(\) | The driver you want to use to send the email | .driver\('mailgun'\) |

## Sending the Notification

Now that we have built our notification above, we can send it in our controller \(or anywhere else we like\):

```python
from app.notifications.WelcomeNotification import WelcomeNotification
from notifications import Notify
...

def show(self, notify: otify):
    notify.mail(WelcomeNotification, to='user@gmail.com')
```

Notice here we simply specified the Notify class in the parameter list and we are able to pass in our awesome new WelcomeNotification into the mail method.

{% hint style="info" %}
NOTE: The method you should use on the notify class should correspond to the method on the notification class. So for example if we want to execute the slack method on the WelcomeNotification then we would call :

```python
notify.slack(WelcomeNotification)
```

The method you call should be the same as the method you want to call on the notification class. The `Notify` class actually doesn't contain any methods but will call the same method on the notification class as you called on the `Notify` class.
{% endhint %}

### Queuing the Notification

If you would like to queue the notification then you just need to inherit the `ShouldQueue` class and it will automatically send your notifications into the queue to be processed later. This is a great way to speed up your application:

```python
from notifications import Notifiable
from masonite.queues import ShouldQueue
import os


class WelcomeNotification(Notifiable, ShouldQueue):

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
```

## Building Our Slack Notification

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

| Method | Description | Example |
| :--- | :--- | :--- |
| .token\(\) | This is your Slack token that has the correct permission scopes. | .token\('xoxp-359926262626-35...'\) |
| .text\(\) | The text you want to show in the message | .text\('Welcome to Masonite!'\) |
| .channel\(\) | The channel you want to broadcast to. If the value you supply starts with a \# sign then Notifications will make a POST request with your token to the Slack channel list API and get the channel ID. You can specify the channel ID directly if you don't want this behavior | .channel\('\#general'\) .channel\('CHSUU862'\) |
| .as\_user\(\) | The username you want to show as the message | .as\_user\('Masonite Bot'\) |
| .icon\(\) | The icon you want to show. This needs to be a Slack emoji | .icon\(':fire:'\) |
| .as\_current\_user\(\) | This sets a boolean value to True on whether the message should show as the currently authenticated user. | .as\_current\_user\(\) |
| .without\_markdown\(\) | This will not parse any markdown in the message. This is a boolean value and requires no parameters. | .without\_markdown\(\) |
| .dont\_unfurl\(\) | This sets a boolean to False on whether the message should show any attachments. Usually slack will show an icon of the website when posting a link. This disables that feature for the current message. | .dont\_unfurl\(\) |
| .as\_snippet\(\) | Used to post the current message as a snippet instead of as a normal message. This option has 3 keyword arguments. The `file_type`, `name`, and `title`.  This uses a different API endpoint so some previous methods may not be used. | .as\_snippet\(file\_type='python', name='snippet', title='Awesome Snippet'\) |
| .comment\(\) | Only used when using the .as\_snippet\(\) method. This will set a comment on the snippet. | .comment\('Great Snippet'\) |
| .button\(\) | Used to create action buttons under a message. This requires `text` and a `url` but can also contain `style`, and `confirm` | .button\('Sure!', '[http://google.com](http://google.com)', style='primary', confirm='Are you sure?'\) |
| .dry\(\) | Sets all the necessary fields but does not actually send the email. This is great for testing purposes. This takes no parameters | .dry\(\) |

## Sending a Slack Notification

Now that we have built our notification above, we can send it in our controller \(or anywhere else we like\):

```python
from app.notifications.WelcomeNotification import WelcomeNotification
from notifications import Notify
...

def show(self, notify: Notify):
    notify.slack(WelcomeNotification)
```

Notice here we simply specified the `Notify` class in the parameter list and we are able to pass in our awesome new WelcomeNotification into the slack method.

## Building Integrations

The `Notifiable` class is very modular and you are able to build custom integrations if you like pretty simply. In this section we'll walk through how to create what are called `Components`.

## What are Components?

Components are classes that can be added to a Notification class that extend the behavior of the notification. In fact, the Notifiable class is just a simple abstraction of two different components. Let's look at the signature of the class that we have been inheriting from.

```python
from notifications.components import MailComponent, SlackComponent


class Notifiable(MailComponent, SlackComponent):
    pass
```

The Component classes are the classes that have our methods we have been using. If you'd like to see the source code on those components you can check them out on GitHub to get a lower level understanding on how they work.

## Creating a Component

Let's walk through a bit on how we created our MailComponent by creating a simplified version of it. First let's create a simple class:

```python
class MailComponent:
    pass
```

Now let's add a line and a subject method to it:

```python
class MailComponent:

    def line(self, message):
        pass

    def subject(self, subject)
        pass
```

and let's use these two methods to build a template attribute

```python
class MailComponent:

    template = ''

    def line(self, message):
        self.template += template
        return self

    def subject(self, subject)
        self._subject = subject
        return self
```

Since we returned self we can keep appending onto the notification class like we have been.

{% hint style="info" %}
The actual MailComponent class is a bit more complex than this but we'll keep this simple for explanatory purposes.
{% endhint %}

### The Fire Method

Whenever we insert the notification into the Notify class:

```python
notify.mail(WelcomeNotification)
```

This will call the mail method on the notification class \(or whatever other method we called on the Notify class\).

Once that is returned then it will call the fire\_mail method which you will specify in your component.

{% hint style="info" %}
If you are created a discord notification then you should have a `fire_discord` method on your component and you will call it using `notify.discord(WelcomeNotification)`.
{% endhint %}

Since we want to call the mail method on it, we will create a `fire_mail` method:

```python
class MailComponent:

    template = ''

    def line(self, message):
        self.template += template
        return self

    def subject(self, subject)
        self._subject = subject
        return self

    def fire_mail(self):
        pass
```

### Passing Data With Protected Members

Sometimes you will want to pass in data into the `fire_mail` method. In order to keep this simple and modular, any keyword arguments you pass into the Notify class will be set on the notification class as a protected member. For example if we have this:

```text
notify.mail(WelcomeNotification, to='admin@site.com')
```

It will set a `_to` attribute on the notification class BEFORE we get to the fire method.

So using the example above we will be able to do:

```python
def fire_mail(self):
    self._to # admin@site.com
```

We can use this behavior to pass in information into the `fire_mail` method while keeping everything clean.

A practical example is sending the message to a certain user:

```text
notify.mail(WelcomeNotification, to='admin@site.com')
```

```python
class WelcomeNotification(Notifiable):

    def mail(self):
        return self.line('{0} We greatly value your service!'.format(self._to)) \
            .action('Sign Back In', href="http://gbaleague.com")
```

Notice here we now have a `_to` member on our class we can use because we passed it through from our `Notify` class.

### Sending The Mail

Ok so finally we have enough information we need to send the actual email. The fire\_method is resolved by the container so we can simply specify what we need to send the email.

Our notification class will look like:

```python
from your.package import MailComponent

class WelcomeNotification(Notifiable, MailComponent):

    def mail(self):
        return self.subject('New account signup!') \
            .line('We greatly value your service!')
```

Our Notify class will look like:

```text
notify.mail(WelcomeNotification, to='admin@site.com')
```

and our fire method will look like:

```python
from masonite import Mail

class MailComponent:

    template = ''

    def line(self, message):
        self.template += template
        return self

    def subject(self, subject)
        self._subject = subject
        return self

    def fire_mail(self, mail: Mail):
        mail.to(self._to) \
            .subject(self._subject) \
            .send(self.template)
```

Remember the `_to` class attribute that came from the keyword argument in the `Notify` class.

