# Creating Commands

## Introduction

It's extremely simple to add commands to Masonite via the craft command tool and Service Providers. If you have been using Masonite for any amount of time you will learn that commands are a huge part of developing web applications with Masonite. We have made it extremely easy to create these commands and add them to craft to build really fast personal commands that you might use often.

Masonite uses the Cleo package for creating and consuming commands so for more extensive documentation on how to utilize commands themselves, how to get arguments and options, and how to print colorful text to the command line.

{% hint style="success" %}
Read more about Cleo by visiting the [Cleo Documentation](http://cleo.readthedocs.io/en/latest/).
{% endhint %}

## Getting Started

You can create commands by using craft itself:

{% code title="terminal" %}
```text
$ python craft command Hello
```
{% endcode %}

This will create a `app/commands/HelloCommand.py` file with boilerplate code that looks like this:

{% code title="app/commands/HelloCommand.py" %}
```python
""" A HelloCommand Command """
from cleo import Command


class HelloCommand(Command):
    """
    Description of command

    command:name
        {argument : description}
    """

    def handle(self):
        pass
```
{% endcode %}

Let's create a simple hello name application which prints "hello your-name" to the console.

Where it says `command:name` inside the docstring we can put `hello` and inside the argument we can put `name` like so:

{% code title="app/commands/HelloCommand.py" %}
```python
""" A HelloCommand Command """
from cleo import Command


class HelloCommand(Command):
    """
    Say hello to you

    hello
        {name : Your name}
    """

    def handle(self):
        pass
```
{% endcode %}

Inside the `handle` method we can get the argument passed by specifying `self.argument('name')`. Simply put:

{% code title="app/commands/HelloCommand.py" %}
```python
""" A HelloCommand Command """
from cleo import Command


class HelloCommand(Command):
    """
    Say hello to you

    hello
        {name : Your name}
    """

    def handle(self):
        print('Hello {0}'.format(self.argument('name')))
```
{% endcode %}

That's it! Now we just have to add it to our craft command.

## Adding Our Command To Craft

We can add commands to craft by creating a Service Provider and registering our command into the container. Craft will automatically run all the register methods on all containers and retrieve all the commands.

Let's create a Service Provider:

{% code title="terminal" %}
```text
$ python craft provider Hello
```
{% endcode %}

This will create a provider in `app/providers/HelloProvider.py` that looks like:

{% code title="app/providers/HelloProvider.py" %}
```python
''' A HelloProvider Service Provider '''
from masonite.provider import ServiceProvider


class HelloProvider(ServiceProvider):

    def register(self):
        pass

    def boot(self):
        pass
```
{% endcode %}

Let's import our command and register it into the container. Also because we are only registering things into the container, we can set `wsgi = False` so it is not ran on every request and only before the server starts:

{% code title="app/providers/HelloProvider.py" %}
```python
''' A HelloProvider Service Provider '''
from masonite.provider import ServiceProvider
from app.commands.HelloCommand import HelloCommand

class HelloProvider(ServiceProvider):

    wsgi = False

    def register(self):
        self.app.bind('HelloCommand', HelloCommand())

    def boot(self):
        pass
```
{% endcode %}

{% hint style="warning" %}
**Make sure you instantiate the command. Also the command name needs to end in "Command". So binding** `HelloCommand` **will work but binding** `Hello` **will not. Craft will only pick up commands that end in** `Command`**. This is also case sensitive so make sure** `Command` **is capitalized.**
{% endhint %}

## Adding The Service Provider

Like normal, we need to add our Service Provider to the `PROVIDERS` list inside our `config/providers.py` file:

{% code title="config/providers.py" %}
```python
from app.providers.HelloProvider import HelloProvider

PROVIDERS = [
...
    # Application Providers
    UserModelProvider,
    MiddlewareProvider,

    # New Hello Provider
    HelloProvider,
]
```
{% endcode %}

That's it! Now if we run:

{% code title="terminal" %}
```text
$ python craft
```
{% endcode %}

We will see our new `hello` command:

{% code title="terminal" %}
```python
  help              Displays help for a command
  hello             Say hello to you
  install           Installs all of Masonite's dependencies
```
{% endcode %}

and if we run:

{% code title="terminal" %}
```text
$ python craft hello Joseph
```
{% endcode %}

We will see an output of:

{% code title="terminal" %}
```text
Hello Joseph
```
{% endcode %}

