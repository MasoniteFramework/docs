# Masonite Triggers

# Introduction

Masonite Triggers is a way to add support for triggering classes within various parts of your project. A great use case is to create a class that sends an email and then simple use `trigger('sendWelcomeEmail')` anywhere in your project.

## Installation

At the root of your Masonite project just run:

```
$ pip install triggers
$ craft publish triggers
```

If publishing does not work, you may be using a virtual environment and may not have the correct `site_packages` directory added to your `config/packages.py` file. Read more about this in the [Publishing Packages](/publishing-packages.md) documentation.

## Usage

The publish command will create a new configuration file under `config/triggers.py` where you can register all of your trigger classes.  To register your class, just enter an alias you’d like to use for your class as the key and then a string with the full module path to the class.

This configuration file may look something like:

```python
TRIGGERS = {
    'sendWelcomeEMail' : 'app.triggers.SendWelcomeEmail.SendWelcomeEmail'
}
```

Lets make this class in `app/triggers/SendWelcomeEmail.py`:

```python
class SendWelcomeEmail(object):

    def __init__(self):
        pass

    def action(self):
        print('Send Welcome Email')
```

By default, all triggers will fire the `action` method on your class.

You may now activate that trigger by using the `trigger()` function:

```python
from triggers.helpers import trigger

trigger('sendWelcomeEmail')
```

All triggers will default to the `action` method. You can specify a different method by calling:

```python
trigger('sendWelcomeEmail@premium')
```

which will call the premium method on the `sendWelcomeEmail` class. If your functions needs additional parameters you can specify them as extra parameters such as:

```python
trigger('sendWelcomeEmail@premium', user, email)
```

A method with that trigger will look like:

```python
class SendWelcomeEmail(object):

    def __init__(self):
        pass

    def action(self):
        print('Send Welcome Email')

    def premium(self, user, email)
        pass
```

That’s it! Triggers are very simple but very powerful.