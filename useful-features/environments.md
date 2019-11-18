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

{% code title="bootstrap/start.py" %}
```python
from masonite.environment import LoadEnvironment
...
LoadEnvironment()
```
{% endcode %}

This class instantiation does a few things:

The first thing is it loads the .env file located in the base of your application into the Python environment. If you installed Masonite using craft install then Masonite automatically create this `.env` file for you based on the `.env-example` file. If you have installed Masonite but do not see this `.env` file then you can create it manually and copy and paste the contents of `.env-example`.

The next thing it will do is look for an `APP_ENV` variable inside your `.env` file it just loaded and then look for an environment with that value.

For example, this variable:

{% code title=".env" %}
```text
...
APP_ENV=local
...
```
{% endcode %}

Will load additionally load the `.env.local` environment file.

This may be useful to have more global environment variables that can be shared across your team like Stripe, Mailgun, or application keys and then have more developer specific values like database connections, Mailtrap or different storage drivers for development.

## Loading Additional Environments

In addition to loading the `.env` file and the additional environment file defined in your `.env` file, you can load a third environment by specifying it in the constructor:

{% code title=".env" %}
```text
...
APP_ENV=local
...
```
{% endcode %}

{% code title="bootstrap/start.py" %}
```python
from masonite.environment import LoadEnvironment
...
LoadEnvironment('development')
```
{% endcode %}

This will load the `.env` file, the `.env.local` file and the `.env.development` environment file. 

## Loading Only A Single Environment

If you don't want to load an additional environment and instead want to load only 1 single environment then you can pass in the `only` parameter.

{% code title="bootstrap/start.py" %}
```python
from masonite.environment import LoadEnvironment
...
LoadEnvironment(only='development')
```
{% endcode %}

This will load only the `.env.development` environment file.

## Getting Environment Variables

Environment variables should be set on a project per project basis inside your .env file. When the server starts, it will load all of those environment variables into the current global environment. You can fetch these environment variables 1 of 2 ways:

### os.getenv

You can obviously get them in the normal Python way by doing something like:

```python
import os

os.getenv('DB_PORT') #== '5432' (string)
```

Notice that the above example is a string. We typically need the data type to be casted to the respective type. For example we need `5432` to be an integer and need `True` to be a boolean.

### masonite.env

We can use the `env()` function in order to accomplish this which takes the place of `os.getenv()`. This looks like:

```python
from masonite import env

env('DB_PORT', 'default') #== 5432 (int)
```

If the value is a numeric then it will cast it to an integer. Below are the examples of what this function will cast:

| Value | Casts to \(type\) |
| :--- | :--- |
| 5432 | 5432 \(int\) |
| true | True \(bool\) |
| True | True \(bool\) |
| false | False \(bool\) |
| False | False \(bool\) |
| smtp | smtp \(string\) |

If you do not wish to cast the value then pass in false as the third parameter:

```python
from masonite import env
​
env('DB_PORT', 'default', cast=False) #== '5432' (string)
```

