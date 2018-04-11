# Creating Commands

# Introduction

It's extremely simple to add commands to Masonite via the craft command tool and Service Providers. If you have been using Masonite for any amount of time you will learn that commands are a huge part of developing web applications with Masonite. We have made it extremely easy to create these commands and add them to craft to build really fast personal commands that you might use often.

## Getting Started

You can create commands by using craft itself:

    $ craft command HelloWorldCommand
    
This will create a `app/commands/HelloWorldCommand.py` file with boiler plate code that looks like this:

```python

""" A HelloWorldCommand Command """
from cleo import Command


class HelloWorldCommand(Command):
    """
    Description of command

    command:name
        {argument : description}
    """

    def handle(self):
        pass
```

Let's create a simple hello world application which prints "hello world" to the console. Inside the `handle` method we can simply put:

```python
...

def handle(self):
    print('Hello World')
```

That's it! Now we just have to add it to our craft command.

## Adding Our Command To Craft






