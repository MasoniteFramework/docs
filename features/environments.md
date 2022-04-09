Environment variables in Masonite are defined in a `.env` file and should contain all environment variables needed for your project.

You can have multiple environment files that are loaded when the server first starts. It is often helpful to have different variable values depending on the environment where the application is running (locally, during tests or on a production server).

Also it might be useful to have more global environment variables that can be shared across your team for 3rd party services like Stripe or Mailgun and then have more developer specific values like database connections or different storage drivers for development.

We'll walk through how to configure your environments in this documentation.

# Environment Security

Environment variables should be set on a project per project basis inside your `.env` file.

{% hint style="warning" %}
Never commit any of your `.env` files into source control ! It would be a security risk in the event someone gained access to your repository since sensitive credentials and data would get exposed.
{% endhint %}

That is why `.env` and `.env.*` are in the project `.gitignore` file by default, so you should not worry about accidentally committing those files to source control.

# Getting Started

In a fresh Masonite installation project root directory will contain an `.env.example` file that defines minimum and common configuration values for a Masonite application. During the installation process, this file will be copied to `.env` file.

If you have installed Masonite but do not see this `.env` file then you can create it manually and copy and paste the contents of `.env-example` file.

# Environments Loading Order

Environment files are loaded in this order:

1. Masonite will load the `.env` file located at your project root into the Python environment.

2. Masonite will look for an `APP_ENV` variable inside the already loaded `.env`. If it is defined it will try to load
the `.env.{APP_ENV}` file corresponding to this environment name.

For example, if `APP_ENV` is set to `local`, Masonite will additionally load the `.env.local` environment file.
{% code title=".env" %}
```bash
APP_ENV=local
```
{% endcode %}

When the server is ready all those variables will be loaded into the current environment ready to be accessed in the different
Masonite configuration files or directly with `env()` helper.

# Getting Environment Variables

## os.getenv()

You can use Python standard `os.getenv()` method to get an environment variable value. It looks like:
```python
import os

is_debug = os.getenv("APP_DEBUG") #== "True" (str)
```
Notice that this method does not cast types, so here we got a string instead of a boolean value.

## env()
You can also use Masonite helper `env` to read an environment variable value. It looks like:
```python
from masonite import env

is_debug = env("APP_DEBUG", False) #== True (bool)
```
Note that you can provide a default value if the environement variable is not defined. Default value is `""`.
For convenience this helper is casting types. Here are different examples of variables type casting:

| Env Var Value | Casts to \(type\) |
| :--- | :--- |
| 5432 | `5432` \(int\) |
| true | `True` \(bool\) |
| | `None` \(None\) |
| "" | `""` \(str\) |
| True | `True` \(bool\) |
| false | `False` \(bool\) |
| False | `False` \(bool\) |
| smtp | `smtp` \(string\) |

If you do not wish to cast the value then you can provide a third parameter `cast`:

```python
from masonite import env
​
env('APP_DEBUG', False, cast=False) #== "False" (str)
```

# Getting Current Environment

The current Masonite environment is defined through the `APP_ENV` variable located in your
`.env` file. You can access it easily through the Masonite app `environment()` helper:

```python
app.environment() #== local
```

When running tests the environment will be set to `testing`.

There are other helpers available regarding environment:
```python

app.is_production() #== True if APP_ENV=production

app.is_running_tests() #== True if running tests
```

# Debug Mode

The debug mode is controlled by the `APP_DEBUG` environment variable used in `config/application.py` configuration file.
When crafting a new project, the debug mode is enabled (`APP_ENV=True`). It should stay enabled for local development.

When debug mode is enabled all exceptions (or routes not found) are rendered as an HTML debug error page containing a lot
of information to help you debug the problem. When disabled, the default `500`, `404`, `403` error
pages are rendered.

You can check if debug mode is enabled through the Masonite app `is_debug()` helper or with the `config` helper:

```python
app.is_debug() #== True

from masonite.configuration import config

config("application.debug") #== True
```

{% hint style="warning" %}
Never deploy an application in production with debug mode enabled ! This could lead to expose some
sensitive configuration data and environment variables to the end user.
{% endhint %}
