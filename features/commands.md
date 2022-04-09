Commands in Masonite are generally designed to make your development life easier. Commands can range from creating controllers and models to installing packages and publishing assets.

Masonite uses the [cleo](https://cleo.readthedocs.io/en/latest/) package for shell command feature.

Available commands for Masonite can be displayed by running:

```terminal
python craft
```

This will show a list of commands already available for Masonite.

Every command has a documentation screen which describes the command's available arguments and options. In
order to view a command documentation, prefix the name of the command with `help`. For example, to see help of
`serve` command you can run:

```terminal
python craft help serve
```

# Creating Commands

Commands can be created with a simple command class inheriting from Masonite `Command` class:

```python
from masonite.commands import Command

class MyCommand(Command):
    """
    Description of Command

    command:signature
        {user : A positional argument for the command}
        {--f|flag : An optional argument for the command}
        {--o|option=default: An optional argument for the command with default value}
    """

    def handle(self):
        pass
```

Command's name, description and arguments are parsed from the Command docstring.

## Name and Description

The docstring should start with the description of the command
```python
"""
Description of Command

"""
```

and then after a blank line you can define the command name.

```python
"""
Description of Command

command_name
"""
```


## Positional Arguments

After the name of the command in the docstring should come the arguments. Arguments are defined with one indent and enclosed into brackets.

Positional (mandatory) arguments are defined without dashes (`-` or `--`).

Here is how to define a positional argument called `name` with a description:
```python
"""
    {name : Description of the name argument}
"""
```

Inside the command, positional arguments can be retrieved with `self.argument(arg_name)`
```python
    def handle(self):
        name = self.argument("name")
```

## Optional Arguments

Optional arguments are defined with dashes and can be used in any order in the command call. An optional
argument can have a short name: `--force` option could have a short name `--f`.

Here is how to define an optional argument called `force` with a description:
```python
"""
command_name
    {--iterations : Description of the iterations argument}
    {--f|force : Description of the force argument}
"""
```

Notice how we provided the short version for the `force` argument but not for the `iterations` arguments

Now the command can be used like this:
```terminal
python craft command_name --f --iterations
python craft command_name --iterations --force
```

If the optional argument is requiring a value you should add the `=` suffix:

```python
"""
command_name
    {--iterations= : Description of the iterations argument}
"""
```

Here when using `iterations`, the user should provide a value.

```terminal
python craft command_name --iterations 3
python craft command_name --iterations=3
```

If the argument may or may not have a value, you can use the suffix `=?` instead.

```python
"""
command_name
    {--iterations= : Description of the iterations argument}
"""
```

```terminal
python craft command_name --iterations
python craft command_name --iterations 3
```

Finally if a default value should be used when no value is provided, add the suffix `={default}`:

```python
"""
command_name
    {--iterations=3: Description of the iterations argument}
"""
```

```terminal
# iterations will be equal to 3
python craft command_name --iterations
# iterations will be equal to 1
python craft command_name --iterations 1
```

Inside the command, optional arguments can be retrieved with `self.option(arg_name)`

```python
    def handle(self):
        name = self.option("iterations")
```

## Printing Messages

You can print messages to console with different formatting:

- `self.info("Info Message")`: will output a message in green
- `self.warning("Warning Message")`: will output a message in yellow
- `self.error("Error Message")`: will output a message in bold red
- `self.comment("Comment Message")`: will output a message in light blue

# Registering Commands

Once created you can register the command to Masonite's [Service Container](../architecture/service-container.md)
inside a [Service Provider](../architecture/service-providers.md) (if you don't have one, you should [create one](../architecture/service-providers.md#creating-a-provider)):

`add()` method takes one or multiple commands:

```python
from some.place.YourCommand import YourCommand

class AppProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.make('commands').add(YourCommand())
```

When you run `python craft` you will now see the command you added.
