# Creating Packages

## Introduction

Creating packages is very simple for Masonite. You can get a package created and on PyPi is less than 5 minutes. With Masonite packages you'll easily be able to integrate and scaffold all Masonite projects with ease. Masonite comes with several helper functions in order to create packages which can add configuration files, routes, controllers, views, commands and more.

## Getting Started

As a developer, you will be responsible for both making packages and consuming packages. In this documentation we'll talk about both. We'll start by talking about how to make a package and then talk about how to use that package or other third party packages.

Masonite, being a Python framework, can obviously utilize all Python packages that aren’t designed for a specific framework. For example, Masonite can obviously use a library like requests but can’t use Django Rest Framework.

Similarly to how Django Rest Framework was built for Django, you can also build packages specific to Masonite. Although you can just as simply build packages for both, as long as you add some sort of Service Provider to your package that can integrate your library into Masonite.

### About Packages

There are several key functions that Masonite uses in order to create applications. These include primarily: routes, controllers, views, and craft commands. Creating a package is simple. Conveniently Masonite comes with several helper functions in order to create all of these.

You can easily create a command like `craft mypackage:install` and can scaffold out and install controllers, routes, etc into a Masonite project.

You do not have to use this functionality and instead have the developer copy and paste things that they need to from your documentation but having a great setup process is a great way to promote developer happiness which is what Masonite is all about.

### Creating a Package

Like other parts of Masonite, in order to make a package, we can use a craft command. The `craft package` command will scaffold out a simple PyPi package and is fully able to be uploaded directly to PyPi.

This should be done in a separate folder outside of your project.

Let's create our package:

```text
$ craft package testpackage
```

This will create a file structure like:

```text
testpackage/
    __init__.py
    integration.py
MANIFEST.in
setup.py
```

### **Creating a Config Package**

Lets create a simple package that will add or append a config file from our package and into the project.

First lets create a config file inside `testpackage/snippets/configs/services.py`. We should now have a project structure like:

```text
testpackage/
    __init__.py
    integration.py
    snippets/
        configs/
            services.py
MANIFEST.in
setup.py
```

Great! inside the `services.py` lets put a configuration setting. This configuration file will be directly added into a Masonite project so you can put doctrings or flagpole comments directly in here:

```text
TESTPACKAGE_PAYMENTS = {
    'stripe': {
        'some key': 'some value',
        'another key': 'another value'
    }
}
```

Perfect! Now we'll just need to tell PyPi to include this file when we upload it to PyPi. We can do this in our `MANIFEST.in` file.

```text
include testpackage/snippets/configs/*
```

### **Creating an Install Command**

It's great \(and convenient\) to add craft commands to a project so developers can use your package more efficiently. You can head over the [Creating Commands](../the-craft-command/creating-commands.md) to learn how to create a command. It only involves a normal command class and a Service Provider.

Head over to that documentation page and create an `InstallCommand` and an `InstallProvider`. This step should take less than a few minutes. Once those are created we can continue to the adding package helpers below.

### **Adding Migration Directories**

Masonite packages allow you to add new migrations to a project. For example, this could be used to add a new `package_subscriptions` table if you are building a package that works for subscribing users to Stripe.

Inside the Service Provider you plan to use for your package we can register our directory:

```python
from masonite.provider import ServiceProvider

package_directory = os.path.dirname(os.path.realpath(__file__))

class ApiProvider(ServiceProvider):

    def register(self):
        self.app.bind(
            'TestPackageMigrationDirectory',
            os.path.join(package_directory, '../migrations')
        )
```

Masonite will find any keys in the container that end with `MigrationDirectory` and will add it to the list of migrations being ran whenever `craft migrate` and `craft migrate:*` commands are ran.

The `package_directory` variable contains the absolute path to the current file so the migration directory being added should also be an absolute path to the migration directory as demonstrated here. Notice the `../migrations` syntax. This is going back one directory and into a migration directory there.

### **Package Helpers**

Almost done. Now we just need to put our `masonite.package` helper functions in our install command. The location we put in our `create_or_append_config()` function should be an absolute path location to our package. To help with this, Masonite has put a variable called `package_directory` inside the `integration.py` file. Our handle method inside our install command should look something like:

```python
import os
from cleo import Command
from masonite.packages import create_or_append_config


package_directory = os.path.dirname(os.path.realpath(__file__))

class InstallCommand(Command):
    """
    Installs needed configuration files into a Masonite project

    testpackage:install
    """

    def handle(self):
        create_or_append_config(
            os.path.join(
                package_directory,
                '../testpackage/snippets/configs/services.py'
            )
        )
```

{% hint style="warning" %}
**Make sure this command is added to your Service Provider and the developer using your package adds it to the** `PROVIDERS` **list as per the** [**Creating Commands**](../the-craft-command/creating-commands.md) **documentation.**
{% endhint %}

This will append the configuration file that has the same name as our package configuration file. In this case the configuration file we are creating or appending to is `config/services.py` because our packages configuration file is `services.py`. If we want to append to another configuration file we can simply change the name of our package configuration file.

### **Working With Our Package**

We can either test our package locally or upload our package to PyPi.

To test our package locally, if you use virtual environments, just go to your Masonite project and activate your virtual environment. Navigate to the folder where you created your package and run:

```text
$ pip install .
```

If you want to be able to make changes without having to constantly reinstall your package then run

```text
$ pip install --editable .
```

This will install your new package into your virtual environment. Go back to your project root so we can run our `craft testpackage:install` command. If we run that we should have a new configuration file under `config/services.py`.

### **Uploading to PyPi**

If you have never set up a package before then you'll need to [check how to make a `.pypirc` file](http://peterdowns.com/posts/first-time-with-pypi.html). This file will hold our PyPi credentials.

To upload to PyPi we just have to pick a great name for our package in the `setup.py` file. Now that you have a super awesome name, we'll just need to run:

```text
$ python setup.py sdist upload
```

which should upload our package with our credentials in our `.pypirc` file. Make sure you click the link above and see how to make once.

If `python` doesn’t default to Python 3 or if PyPi throws errors than you may need to run:

```text
$ python3 setup.py sdist upload
```

### **Consuming a package.**

Now that your package is on PyPi we can just run:

```text
$ pip install super_awesome_package
```

Then add your Service Provider to the `PROVIDERS` list:

```python
PROVIDERS = [
    ...
    # New Provider
    'testpackage.providers.InstallProvider.InstallProvider',
]
```

and then run:

```text
$ craft testpackage:install
```

Remember our Service Provider added the command automatically to craft.

Again, not all packages will need to be installed or even need commands. Only packages that need to scaffold the project or something similar need one. This should be a judgment call on the package author instead of a requirement.

You will know if a package needs to be installed by reading the packages install documentation that is written by the package authors.

## Helper Functions

These helper functions are used inside the install commands or anywhere else in your package where you need to scaffold a Masonite project.

The `location` specified as parameters here are absolute path locations. You can achieve this by using the `package_directory` variable in your `integration.py` file.

To achieve an absolute path location, this will look like:

```python
location = os.path.join(
            package_directory, 'snippets/configs/services.py'
        )
```

All helper functions are located in the `masonite.packages` module. To use these functions you’ll need to import the function to be used like:

```python
from masonite.packages import create_or_append_config
```

### **Creating Configuration Files**

`create_or_append_config(location)` will create a configuration file based on a configuration file from your package.

### **Creating Web Routes**

`append_web_routes(location)` will append web routes to the `routes/web.py` file. Your web routes should have a `+=` to the `ROUTES` constant and should look something like:

```python
ROUTES += [
    # your package routes here
]
```

### **Creating Api Routes**

`append_api_routes(location)` will append api routes to a masonite project under `routes/api.py`. Your api routes should have a `+=` to the `ROUTES` constant and should look something like:

```python
ROUTES += [
    # your package routes here
]
```

### **Creating Controllers**

`create_controller(location)` will take a controller from your package and append it under the `app.http.controllers` namespace by default.

You can also optionally add a `to` parameter to specify which directory you want to install the controller into. The directory will automatically be created.

```python
create_controller(
    os.path.join(
        package_directory, 'snippets/controller/ControllerHere.py'
    ),
    to = 'app/http/controllers/Vendor/Package'
)
```

This will create the controller in `app.http.controller.Vendor.Package.ControllerHere.py`

