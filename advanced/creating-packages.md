# Creating Packages

## Introduction

Creating packages is very simple for Masonite. You can get a package created and on PyPi is less than 5 minutes. With Masonite packages you'll easily be able to integrate and scaffold all Masonite projects with ease. Masonite comes with several helper functions in order to create packages which can add configuration files, routes, controllers, views, commands, migrations and more.

## Getting Started

As a developer, you will be responsible for both making packages and consuming packages. In this documentation we'll talk about both. We'll start by talking about how to make a package and then talk about how to use that package or other third party packages.

Masonite, being a Python framework, can obviously utilize all Python packages that aren’t designed for a specific framework. For example, Masonite can obviously use a library like requests but can’t use Django Rest Framework.

Similarly to how Django Rest Framework was built for Django, you can also build packages specific to Masonite. You also don't need to build for one use case or the other. You can easily build a Python package that can be used specifically for any Python package but also create a way it can wire directly into Masonite using Service Providers. We'll talk more about this later.

## About Packages

There are several key functions that Masonite uses in order to create applications. These include primarily: routes, controllers, views, migrations, and craft commands. Creating a package is simple. Conveniently Masonite comes with several helper functions in order to create all of these.

You can easily create a command like `craft mypackage:install` and can scaffold out and install controllers, routes, etc into a Masonite project. You can also use the publish command which will look something like `craft publish YourServiceProvider`. There really isn't much of a difference in terms of functionality but the install command will require you to manually copy things to where they need to go and the built in `publish` command takes care of some of these things for you.

You do not have to use this functionality and instead have the developer copy and paste things that they need to from your documentation but having a great setup process is a great way to promote developer happiness which is what Masonite is all about.

## Creating a Package

The best way to create a package is to download the [Masonite Starter Package Repo](https://github.com/MasoniteFramework/starter-package).

Just go to the repo and download a zip of the file. It's not beneficial to simply git clone it since you will eventually need to create your own repository and cloning the repo will still track the existing starter-package repo.

Once downloaded you can unzip it to wherever you want on your machine. From there you should create a virtual environment and run the tests:

### Create Your Virtual Environment

```text
$ python3 -m venv venv
```

### Activate Your Virtual Environment

Activating your virtual environment on Mac and Linux is simple:

{% code title="terminal" %}
```text
$ source venv/bin/activate
```
{% endcode %}

or if you are on Windows:

{% code title="terminal" %}
```text
$ ./venv/Scripts/activate
```
{% endcode %}

### Install Requirements

Now you can install some development pypi packages to help you in your package development journey like `flake8` and `pytest`.

The starter package comes with a make file to help you get started faster. Just run:

```text
$ make init
```

This will install the craft CLI tool as well as some other requirement packages.

### Run the tests

Now you can run the tests to make sure everything is working properly:

```text
$ python -m pytest
```

You should see all the basic setup tests passing. Now you can start your TDD flow or start building tests around your package.

The testing suite is the full Masonite testing suite so be sure to read the Testing docs.

## Building an Install Command

First we will walk through how to create a simple install command. This will show you how to move things manually into the correct locations. After, we will look into using the `publish` command which will automate some of these things. If you want to skip to that part you can scroll down a bit to the next section.

It's great \(and convenient\) to add craft commands to a project so developers can use your package more efficiently. You can head over to [Creating Commands](../the-craft-command/creating-commands.md) to learn how to create a command. It only involves a normal command class and a Service Provider.

Head over to that documentation page and create an `InstallCommand` and an `InstallProvider`. This step should take less than a few minutes. Once those are created we can continue to the adding package helpers below.

Remember you have access to craft commands so you can do something like:

```text
$ craft command Install
$ craft provider Package
```

You'll need to move your command inside the `src/package` directory but it will prevent you from having to write a lot of boiler plate while developing your package.

Just move these generated files into the `src/package/commands` and `src/package/providers` directories. You may have to create these directories if they don't exist.

### Adding Migration Directories

Masonite packages allow you to add new migrations to a project. For example, this could be used to add a new `package_subscriptions` table if you are building a package that works for subscribing users to Stripe.

Inside the Service Provider you plan to use for your package we can register our directory:

```python
import os

from masonite.provider import ServiceProvider

package_directory = os.path.dirname(os.path.realpath(__file__))

class PackageProvider(ServiceProvider):

    def register(self):
        self.app.bind(
            'PackageMigrationDirectory',
            os.path.join(package_directory, '../migrations')
        )
```

Masonite will find any keys in the container that end with `MigrationDirectory` and will add it to the list of migrations being ran whenever `craft migrate` and `craft migrate:*` commands are ran. When you run migrations you will now see something like this:

```text
Migrating: databases/migrations 

Migrating: /Users/joseph/Programming/packages/starter/src/package/providers/../migrations 

[OK] Migrated 2019_12_31_041847_create_package_table
```

Notice how we are now migrating several directories at a time rather than only from a single directory. If this is the approach you want your package to take then this is the best way to do it.

The `package_directory` variable contains the absolute path to the current file so the migration directory being added should also be an absolute path to the migration directory as demonstrated here. Notice the `../migrations` syntax. This is going back one directory and into a migration directory there.

### Package Helpers

There are a few package helpers you can use to integrate your package into a Masonite application. Now we just need to put our `masonite.package` helper functions in our install command. The location we put in our `create_or_append_config()` function should be an absolute path location to our package.

Let's create a basic config file and then have the install command copy it over when they run the command. Let's create a simple file in `src/package/snippets/configs/services.py`

```text
"""Config File."""

DRIVER = 'service1'
```

To help with this, Masonite has put a variable called `package_directory` here. Our handle method inside our install command should look something like:

```python
import os
from cleo import Command
from masonite.packages import create_or_append_config

package_directory = os.path.dirname(os.path.realpath(__file__))


class InstallCommand(Command):
    """
    Installs needed configuration files into a Masonite project

    package:install
    """

    def handle(self):
        create_or_append_config(
            os.path.join(
                package_directory,
                '../snippets/configs/services.py'
            )
        )
```

Now when you run the install command it will create or append the config to the `config/services.py` file.

{% hint style="warning" %}
**Make sure this command is added to your Service Provider and the developer using your package adds it to the** `PROVIDERS` **list as per the** [**Creating Commands**](../the-craft-command/creating-commands.md) **documentation.**
{% endhint %}

This will append the configuration file that has the same name as our package configuration file. In this case the configuration file we are creating or appending to is `config/services.py` because our packages configuration file is `services.py`. If we want to append to another configuration file we can simply change the name of our package configuration file.

## Working With Our Package

We can either test our package locally or upload our package to PyPi.

To test our package locally, if you use virtual environments, just go to your Masonite project and activate your virtual environment. Navigate to the folder where you created your package and run:

```text
$ pip install .
```

If you make changes you'll need to uninstall and reinstall the package buy running something like:

```text
$ pip uninstall your-package && pip install .
```

This will install your new package into your virtual environment. Go back to your project root so we can run our `craft package:install` command. If we run that we should have a new configuration file under `config/services.py`.

## Uploading to PyPi

So we have our package installable as well as tested and integrated nicely into an existing Masonite app. Now it's time to upload it to PYPI so everyone can install it.

We made it really easy to do this with the make commands.

## .pypirc File

If you have uploaded PYPI packages before you'll know you need a file in your home directory.

If you do not have one then you go create one using another make command:

```text
make pypirc
```

This will move the file to your home directory. If you are using windows you may need to move this file manually.

Make sure you go to your `setup.py` file and change all the package configurations. There are lots of comments next to each option so if you are unsure what it does then just give it a read.

Once you have your setup.py file looking correct you can run the make command:

```text
$ make publish
```

This will install twine if it doesn't exist, build the package, upload it to PYPI and delete the artifacts.

Easy, right?

#### Consuming a package.

Now that your package is on PyPi we can just run:

```text
$ pip install super-awesome-package
```

Then add your Service Provider to the `PROVIDERS` list:

```python
from package.providers.InstallProvider import InstallProvider

PROVIDERS = [
    ...
    # New Provider
    InstallProvider,
]
```

and then run:

```text
$ craft package:install
```

Remember our Service Provider added the command automatically to craft.

Again, not all packages will need to be installed or even need commands. Only packages that need to scaffold the project or something similar need one. This should be a judgment call on the package author instead of a requirement.

You will know if a package needs to be installed by reading the packages install documentation that is written by the package authors.

## Publishing

Masonite has the concept of publishing packages. This allows you to manage the integration with your package and Masonite in a more seamless way. Publishing allows you to add things like routes, views, migrations and commands easily into any Masonite app and it is all handled through your service provider

The goal is to have a developer run:

```bash
$ craft publish YourProvider
```

This should be the name of your provider class.

and have all your assets moved into the new Masonite application.

### Publishing Files

You can create or append any files you need to in a developers masonite application. This can be used for any files to include commands, routes, config files etc.

For example let's say you have a directory in your package like:

```text
validation/
  providers/
    ValidationProvider.py
  commands/
    RuleCommand.py
setup.py
```

Inside our service provider we can do this:

```python
import os

class ValidationProvider(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self):
        command_path = os.path.join(os.path.dirname(__file__), '../commands')

        self.publishes({
            os.path.join(command_path, 'RuleCommand.py'): 'app/commands/RuleCommand.py'
        })
```

Notice our command path is 1 directory back inside the `commands` directory. We then combine the directory with the `RuleCommand.py` file and tell Masonite to put it inside the `app/commands/RuleCommand.py` module inside the users directory.

The user of your package will now have a new command in their application!

### Publishing Migrations

You can take any migrations in your package and send them to the Masonite applications migration directory. This is useful if you want to have some developers edit your custom migrations before they migrate them.

For example let's say you have a directory in your package like:

```text
validation/
  providers/
    ValidationProvider.py
  migrations/
    user_migration.py
    team_migration.py
setup.py
```

The migrations like `user_migration.py` should be full migration files.

Then you can have a service provider like this:

```python
import os

def boot(self):
    migration_path = os.path.join(os.path.dirname(__file__), '../migrations')

    self.publishes_migrations([
        os.path.join(migration_path, 'user_migration.py'),
        os.path.join(migration_path, 'team_migration.py'),
    ])
```

This will create a new migration in the users directory.

### Publishing Tags

You can also add tags to each of these migrations as well. For example if you have 2 sets of migrations you can do this instead:

```python
import os

def boot(self):
    migration_path = os.path.join(os.path.dirname(__file__), '../migrations')
    command_path = os.path.join(os.path.dirname(__file__), '../commands')

    self.publishes({
        os.path.join(command_path, 'RuleCommand.py'): 'app/commands/RuleCommand.py'
    }, tag="commands")

    self.publishes_migrations([
        os.path.join(migration_path, 'user_migration.py'),
        os.path.join(migration_path, 'team_migration.py'),
    ], tag="migrations")
```

Now a user can only either publish migrations or commands by adding a `--tag` option

```bash
$ craft publish ValidationProvider --tag migrations
```

This will ignore the commands publishing and only publish the migrations

