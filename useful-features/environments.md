# Environments

## Introduction

Environments in Masonite are defined in `.env` files and contain all your secret environment variables that should not be pushed into source control. You can have multiple environment files that are loaded when the server first starts. We'll walk through how to configure your environment variables in this documentation.

{% hint style="warning" %}
Never load any of your .env files into source control. `.env` and `.env.*` are in the `.gitignore` file by default so you should not worry about accidentally pushing these files into source control.
{% endhint %}

## Getting Started

Masonite comes with a `LoadEnvironment` class that is called in the `bootstrap/start.py` file. This file in imported into the `wsgi.py` file which is where the execution of the environment actually happens because of the import. 

You likely won't have to use this class since this class handles most use cases by default but we will go over how the class itself works.

In `bootstrap/start.py` you will see a code that looks something like:

{% code-tabs %}
{% code-tabs-item title="bootstrap/start.py" %}
```python
from masonite.environment import LoadEnvironment
...
LoadEnvironment()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This class instantiation does a few things:

The first thing is it loads the .env file located in the base of your application into the Python environment. If you installed Masonite using craft install then Masonite automatically create this `.env` file for you based on the `.env-example` file. If you have installed Masonite but do not see this `.env` file then you can create it manually and copy and paste the contents of `.env-example`.

The next thing it will do is look for an `APP_ENV` variable inside your `.env` file it just loaded and then look for an environment with that value.

For example, this variable:

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
...
APP_ENV=local
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Will load additionally load the `.env.local` environment file.

This may be useful to have more global environment variables that can be shared across your team like Stripe, Mailgun, or application keys and then have more developer specific values like database connections, Mailtrap or different storage drivers for development.

## Loading Additional Environments

In addition to loading the `.env` file and the additional environment file defined in your `.env` file, you can load a third environment by specifying it in the constructor:

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
...
APP_ENV=local
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="bootstrap/start.py" %}
```python
from masonite.environment import LoadEnvironment
...
LoadEnvironment('development')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will load the `.env` file, the `.env.local` file and the `.env.development` environment file. 

## Loading Only A Single Environment

If you don't want to load an additional environment and instead want to load only 1 single environment then you can pass in the `only` parameter.

{% code-tabs %}
{% code-tabs-item title="bootstrap/start.py" %}
```python
from masonite.environment import LoadEnvironment
...
LoadEnvironment(only='development')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will load only the `.env.development` environment file.





