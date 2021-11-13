# Introduction

Creating packages is very simple for Masonite. You can create a package and publish it on PyPi in less than 5 minutes. With Masonite packages you will scaffold and centralize some features to reuse it all your Masonite projects with ease. Masonite comes with several helper functions in order to create packages which can add configuration files, routes, controllers, views, commands, migrations and more.

As a developer, you will be responsible for both making packages and consuming packages. In this documentation we'll talk about both. We'll start by talking about how to make a package and then talk about how to use that package or other third party packages.

Masonite, being a Python framework, you can obviously use all Python packages that aren’t designed for a specific framework. For example, you can obviously use a library like `requests` but you can’t use specific Django Rest Framework.

# Package Discovery

Package providers are the connection between your package and Masonite. A service provider is responsible for binding things into Masonite container and specifying from where to load package resources such as views, configuration, routes and assets.

Your Masonite project will discover packages through the `PROVIDERS` list defined in your `providers.py` configuration file. When a package provider is added to this list, this will allow additional bindings, commands, views, routes, migrations and assets to be registered in your project.

Keep in mind that some simple packages do not need to register resources in your project though.

# Creating a Package

## Package Scaffolding

We provide two ways to quickly scaffold your package layout:

- [starter-package](https://github.com/MasoniteFramework/starter-package) GitHub template
- [cookiecutter template](https://github.com/girardinsamuel/cookiecutter-masonite-package/)

### starter-package

The [starter-package](https://github.com/MasoniteFramework/starter-package) is just a GitHub template so you only have to click `Use this template` to create your own GitHub repository scaffolded with the default package layout and then clone your repository to start developing locally.

### cookiecutter template

The [cookiecutter template](https://github.com/girardinsamuel/cookiecutter-masonite-package/) is using `cookiecutter` package to scaffold your package with configuration options (name, author, url...). The advantages is that you won't need to edit all the occurences of your package name after generation.

Install `cookiecutter` globally (or locally) on your computer:

```bash
$ pip install cookiecutter
```

Then you just have to run (in the directory where you want your package repository to be created):

```bash
$ cookiecutter https://github.com/girardinsamuel/cookiecutter-masonite-package.git
```

You can now start developing your package !

## Development process

### Creating and activating your virtual environment

There are man ways to create a virtual environment in Python but here is a simple way to do it:

```bash
$ python3 -m venv venv
```

Activating your virtual environment on Mac and Linux is simple:

```bash
$ source venv/bin/activate
```

or if you are on Windows:

```bash
$ ./venv/Scripts/activate
```

### Initializing your environment

The default package layout contains a Makefile that help getting started quickly. You just have to run:

```bash
$ make init
```

This will install Masonite, development tools such as `flake8` and `pytest` and it will also install your package locally so that you can start using and testing it in the test project directly.

### Running the tests

Now you can run the tests to make sure everything is working properly:

```bash
$ python -m pytest
```

The default package layout come with one basic test. You should see this test passing. You can then start building your package and adding more unit tests.

### Running the test project

The default package layout comes with a test project located in `tests/integrations`. This project is really useful to directly test your package behaviour. It is scaffolded as a default Masonite project with your package already installed (see [Installing a Package](#installing-a-package) section). You can run the project with the usual command:

```bash
$ python craft serve
```

You can then visit `http://localhost:8000` to see the welcome page.

{% hint style="info" %}
When you make changes to your package as your package is installed locally and registered into your project, your changes will be directly available in the project. You will just need to refresh your pages to see changes.
{% endhint %}

### Releasing the package

Once you're satisfied with your package it's time to release it on [PyPi](https://pypi.org/) so that everyone can install it. We made it really easy to do this with the make commands.

#### Registering on PyPi

If you never uploaded a package on PyPi before you will need to [register](https://pypi.org/account/register/). Verify your email address.

#### Creating a publish token

API tokens provide an alternative way (instead of username and password) to authenticate when uploading packages to PyPI. It is is strongly recommended to use an API token where possible.

Go to your PyPi account and find the `API tokens` section. Click on `Add API token`, give a significant name such as `publish-token` and create the token. Write down the token key somewhere for later.

#### Configuring PyPi on your computer

If you never uploaded a package on PyPi before you will need to configure PyPi on your computer. For this you will need to add a `.pypirc` file in your home directory. If you do not have one then you can easily creating one using:

```bash
$ make pypirc
```

This will move the file to your home directory. If you are using Windows you may need to move this file manually.

Then fill in `password` key with the token you created later prefixed by `pypi-`. With a token starting with `AgEIcHlwaS5vcmcCJGNjYjA4M...` the `.pypirc` file would look like:

```
[distutils]
index-servers =
  pypi
  pypitest

[pypi]
username=__token__
password=pypi-AgEIcHlwaS5vcmcCJGNjYjA4M...
```

#### Publishing the package to PyPI

Now you're ready to upload your package to PyPI. Ensure that all parameters of `setup.py` file are up to date. It's the file describing your package. Fill in the correct version number you want to publish. When ready you can run:

```bash
$ make publish
```

This will install `twine` if not installed yet, build the package, upload it to PyPi and delete the build artifacts. You should then see a success message and be able to browse your package on [PyPi](https://pypi.org/).

{% hint style="warning" %}
You should always check that the package name is available on PyPi and that the version number to publish has not been published before. Else you won't be able to publish your package.
{% endhint %}

## Registering Resources

When developing a package you might need to use a configuration file, to add migrations, routes and controllers or views. All those resources can be located in your package but at one point a user might want to override it and will need to publish those resources locally in its project.

The following section will explain how to register those resources in your package to be used in a Masonite project and how to make those resources publishable.

Masonite makes it really easy to create a Package provider that will register your package resources.

The default package comes with a provider file where a provider class inheriting from `PackageProvider` is defined:

```python
# providers/SuperAwesomeProvider.py
from masonite.packages import PackageProvider

class SuperAwesomeProvider(PackageProvider):

    def configure(self):
        (
            self.root("src/super_awesome_package")
            .name("super_awesome")
        )

    def boot(self):
        pass
```

`configure()` method is called in usual `register()` method and is used to register all resources used in your package.

TODO explain name and root

#### root()

#### name()

### Configuration

Your package is likely to have a configuration file. You will want to make your package configuration available through the handy `config()` Masonite helper. For this you will need to call `config(path, publish=False)` inside `configure()` method:

```python
def configure(self):
    (
        self.root("src/super_awesome_package")
        .name("super_awesome")
        .config("config/super_awesome.py")
    )
```

This will load the package configuration file located at `src/super_awesome_package/config/super_awesome.py` into Masonite config. The configuration will then be available with `config("super_awesome.key")`.

If you want to allow users to publish the configuration file into their own project you should add `publish=True` argument.

```python
def configure(self):
    (
        self.root("src/super_awesome_package")
        .name("super_awesome")
        .config("config/super_awesome.py", publish=True)
    )
```

The [package publish]() command will publish the configuration into the defined project configuration folder. With the default project settings it would be in `config/super_awesome.py`.

{% hint style="info" %}
Configuration values located in packages and in local project will be merged. Values defined locally in the project takes precedance over the default values of the package.
{% endhint %}

### Migrations

If your package contains migrations you can register the migration files to be published in a project:

```python
def configure(self):
    (
        self.root("src/super_awesome_package")
        .name("super_awesome")
        .migrations("migrations/create_some_table.py", "migrations/create_other_table.py")
    )
```

The [package publish]() command will publish the migrations files into the defined project migrations folder. With the default project settings it would be in `databases/migrations/`. Migrations file are published with a timestamp, so here it would result in those two files: `{timestamp}_create_some_table.py` and `{timestamp}_create_other_table.py`.

### Routes / Controllers

### Views

### Assets

### Commands

## Publishing Resources

# Consuming a Package

If the package has been released on PyPi you need to install it as any Python package:

```bash
$ pip install super-awesome-package
```

Then you should **follow package installation guidelines** but often it will consist in:

- registering the package [Service Provider](#) in your project:

```python
from super_awesome_package.providers import SuperAwesomeProvider

PROVIDERS = [
  # ...
  SuperAwesomeProvider,
]
```

- publishing some files if you need to tweak package resources or configuration:

```bash
$ python craft package:publish super-awesome-package
```

You should be ready to use the package in your project !
