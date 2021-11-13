# Introduction

Creating packages is very simple for Masonite. You can create a package and publish it on PyPi in less than 5 minutes. With Masonite packages you will scaffold and centralize some features to reuse it all your Masonite projects with ease. Masonite comes with several helper functions in order to create packages which can add configuration files, routes, controllers, views, commands, migrations and more.

As a developer, you will be responsible for both making packages and consuming packages. In this documentation we'll talk about both. We'll start by talking about how to make a package and then talk about how to use that package or other third party packages.

Masonite, being a Python framework, you can obviously use all Python packages that aren’t designed for a specific framework. For example, you can obviously use a library like `requests` but you can’t use specific Django Rest Framework.

# Package Discovery

Explain service providers ...

# Creating a Package

## Package Scaffolding

Two ways to quickly scaffold your package layout:

- cookiecutter
- github repo template

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

## Resources

When developing a package you might need to use a configuration file, to add migrations, routes and controllers or views. All those resources can be located in your package but at one point a user might want to override it and will need to publish those resources locally in its project.

The following section will explain how to register those resources in your package to be used in a Masonite project and how to make those resources publishable.

### Configuration

### Migrations

### Routes / Controllers

### Views

### Assets

## Commands

# Installing a Package

# Publishing a Package
