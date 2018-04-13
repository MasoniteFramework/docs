# Creating Packages

## Creating a Masonite Package

## Introduction

Creating packages is very simple for Masonite. You can get a package created and on PyPi is less than 5 minutes. With Masonite packages you'll easily be able to integrate and scaffold all Masonite projects with ease. Masonite comes with several helper functions in order to create packages that can add configuration files, routes, controllers, views and commands.

### Getting Started

As a developer, you will be responsible for both making packages and consuming packages. In this documentation we'll talk about both. We'll start by talking about how to make a package and then talk about how to use that package or other third party packages.

Masonite, being a Python framework, can obviously utilize all Python packages that aren’t designed for a specific framework. For example, Masonite can obviously use a library like requests but can’t use Django Rest Framework.

Similarly to how Django Rest Framework was built for Django, you can also build packages specific to Masonite.

#### About Packages

There are several key functions that Masonite uses in order to create applications. These include primarily: routes, controllers, views, and craft commands. Creating a package is simple and conveniently Masonite comes with several helper functions in order to create all of these. The developer using your package just needs to run `craft publish your-package-name` and your package can scaffold out their application for them.

You do not have to use this functionality and instead have the developer copy and paste things that they need but having great setup process is a great way to promote developer happiness.

#### Creating a Package

Like other parts of Masonite, in order to make a package, we can use a craft command. The `craft package` command will scaffold out a simple Masonite package and is fully able to be uploaded directly to PyPi.

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

The `integration.py` file is important and should not be removed. This is the file that will be used when our users use the `craft publish` command.

If we open this file we'll notice a single `boot()` function. Whenever the user \(the developer using Masonite\) uses the `craft publish testpackage` command, craft will execute `testpackage.integration.boot()` so it's wise to load anything you want to be executed on in this function.

You'll notice a helper function imported for you at the top. This `create_or_append_config()` function does exactly what it says. It will take a config file from your package and put it into the project by either creating it \(if it does not exist\) or appending it \(if it does exist\). We'll talk about helper functions later on.

**Creating a Config Package**

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

Great! inside the `services.py` lets put a configuration setting:

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

Almost done. Now we just need to put our `masonite.package` helper function in our boot file. The location we put in our `create_or_append_config()` function should be an absolute path location to our package. To do this, Masonite has put a variable called `package_directory` inside our `integrations.py` file. Our boot method should look something like:

```python
def boot():
    create_or_append_config(
        os.path.join(
            package_directory, 'snippets/configs/services.py'
        )
    )
```

This will append the configuration file that has the same name as our package configuration file. In this case the configuration file we are creating or appending to is `config/services.py` because our packages configuration file is `services.py`. If we want to append to another configuration file we can simply change the name of our package configuration file.

**Working With Our Package**

We can either test our package locally or upload our package to PyPi.

To test our package locally, if you use virtual environments, just go to your Masonite project and activate your virtual environment. Navigate to the folder where you created your package and run:

```text
$ pip install .
```

If you want to be able to make changes without having to constantly reinstall your package then run

```text
$ pip install --editable .
```

This will install your new package into your virtual environment. Go back to your project root so we can run our `craft publish` command. If we run `craft publish testpackage` we should get a module not found error.

It's important to note that `craft publish` does not have access to your virtual environment by default, only your system packages. What we can do is add our virtual environments `site_packages` directory to our `config/packages.py` config file which should look something like:

```python
SITE_PACKAGES = [
    'venv/lib/python3.6/site-packages'
]
```

This will take that path and add it to the `sys.path` for the `craft publish` script.

Now if we run `craft publish` we should see that our new configuration file is now in `config/services.py` file. Awesome! We tried making package support super easy. You of course don't need to integrate directly or scaffold the project but the option is there if you choose to make a package to do so.

**Uploading to PyPi**

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

**Consuming a package.**

Now that your package is on PyPi we can just run:

```text
$ pip install super_awesome_package
$ craft publish super_awesome_package
```

Again, not all packages will need to be published. Only packages that need to scaffold the project. You will know if a package needs to be published by reading the packages install documentation.

### Helper Functions

These helper functions are used inside the `boot` function and are only needed when you need your package to be published.

The `location` specified as parameters here are absolute path locations. You can achieve this by using the `package_directory` variable in your `integration.py` file.

To achieve an absolute path location, this will look like:

```python
location = os.path.join(
            package_directory, 'snippets/configs/services.py'
        )
```

#### Functions

All helper functions are located in the `masonite.packages` module. To use these functions you’ll need to import the function to be used like:

```python
from masonite.packages import create_or_append_config
```

`create_or_append_config(location)` will create a configuration file based on a configuration file from your package.

`append_web_routes(location)` will append web routes to the `routes/web.py` file. Your web routes should have a `+=` to the `ROUTES` constant and should look something like:

```python
ROUTES += [
    # your package routes here
]
```

`append_api_routes(location)` will append api routes to a masonite project under `routes/api.py`. Your api routes should have a `+=` to the `ROUTES` constant and should look something like:

```python
ROUTES += [
    # your package routes here
]
```

`create_controller(location)` will take a controller from your package and append it under the `app.http.controllers` namespace.

