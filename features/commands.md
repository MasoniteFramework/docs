Commands in Masonite are generally designed to make your development life easier. Commands can range from creating controllers and models to installing packages and publishing assets.

Masonite uses the `cleo` package for it's command features.

Commands for Masonite can be seen by running:

```
$ python craft
```

This will show a list of commands already available for Masonite.

Every command has a "help" screen which describes the command's available arguments and options. In
order to view the help prefix the name of the command with `help`. For example, to see help of
`serve` command you can run:

```
$ python craft help serve
```

# Creating Commands

Commands can be created a simple basic command class and inheriting cleo's `Command` class:

```python
from cleo import Command

class KeyCommand(Command):
    """
    Description of Command

    command:signature
        {--f|--flag : A flag you can use for the command}
        {--o|--option=default: An option for the command}
    """

    def handle(self):
        pass
```

Once created you can register the command to Masonite's [Service Container](../architecture/service-container.md) so it will show up in show up when you run `python craft`

# Registering Commands

To register command you will need to register them against Masonite's [Service Container](../architecture/service-container.md). You can do so in a service provider

```python
from some.place.YourCommand import YourCommand

class AuthenticationProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.make('commands').add(YourCommnand())
```

When you run `python craft` you will now see the command you added.
