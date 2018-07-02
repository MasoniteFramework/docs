# Creating Commands

## Introduction

It's extremely simple to add commands to Masonite via the craft command tool and Service Providers. If you have been using Masonite for any amount of time you will learn that commands are a huge part of developing web applications with Masonite. We have made it extremely easy to create these commands and add them to craft to build really fast personal commands that you might use often.

Masonite uses the Cleo package for creating and consuming commands so for more extensive documentation on how to utilize commands themselves, how to get arguments and options, and how to print colorful text to the command line.

{% hint style="success" %}
Read more about Cleo by visiting the [Cleo Documentation](http://cleo.readthedocs.io/en/latest/).
{% endhint %}

## Getting Started

You can create commands by using craft itself:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft command HelloCommand
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create a `app/commands/HelloCommand.py` file with boiler plate code that looks like this:

{% code-tabs %}
{% code-tabs-item title="app/commands/HelloCommand.py" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

Let's create a simple hello name application which prints "hello your-name" to the console. 

Where it says `command:name` inside the docstring we can put `hello` and inside the argument we can put `name` like so:

{% code-tabs %}
{% code-tabs-item title="app/commands/HelloCommand.py" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

Inside the `handle` method we can get the argument passed by specifying `self.argument('name')`. Simply put:

{% code-tabs %}
{% code-tabs-item title="app/commands/HelloCommand.py" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

That's it! Now we just have to add it to our craft command.

## Adding Our Command To Craft

We can add commands to craft by creating a Service Provider and registering our command into the container. Craft will automatically run all the register methods on all containers and retrieve all the commands.

Let's create a Service Provider:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft provider HelloProvider
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create a provider in `app/providers/HelloProvider.py` that looks like:

{% code-tabs %}
{% code-tabs-item title="app/providers/HelloProvider.py" %}
```python
''' A HelloProvider Service Provider '''
from masonite.provider import ServiceProvider


class HelloProvider(ServiceProvider):

    def register(self):
        pass

    def boot(self):
        pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Let's import our command and register it into the container. Also because we are only registering things into the container, we can set `wsgi = False` so it is not ran on every request and only before the server starts:

{% code-tabs %}
{% code-tabs-item title="app/providers/HelloProvider.py" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="warning" %}
**Make sure you instantiate the command. Also the command name needs to end in "Command". So binding** `HelloCommand` **will work but binding** `Hello` **will not. Craft will only pick up commands that end in** `Command`**. This is also case sensitive so make sure** `Command` **is capitalized.**
{% endhint %}

## Adding The Service Provider

Like normal, we need to add our Service Provider to the `PROVIDERS` list inside our `config/application.py` file:

{% code-tabs %}
{% code-tabs-item title="config/application.py" %}
```python
PROVIDERS = [
...
    # Application Providers
    'app.providers.UserModelProvider.UserModelProvider',
    'app.providers.MiddlewareProvider.MiddlewareProvider',

    # New Hello Provider
    'app.providers.HelloProvider.HelloProvider',
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

That's it! Now if we run:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We will see our new `hello` command:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```python
  help              Displays help for a command
  hello             Say hello to you
  install           Installs all of Masonite's dependencies
```
{% endcode-tabs-item %}
{% endcode-tabs %}

and if we run:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft hello Joseph
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We will see an output of:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
Hello Joseph
```
{% endcode-tabs-item %}
{% endcode-tabs %}

