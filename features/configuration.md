Configuration files in Masonite are gathered in one folder named `config` in default projects.

Each feature can have some options in its own file named after the feature. For example you will find mail related options in `config/mail.py` file.


# Getting Started

The `Configuration` class is responsible for loading all configuration files before the application starts.

It will load all files located at path defined through `config.location` binding which default to `config/`.

Then values are accessed based on the file they belong to and a dotted path can be used to access nested options.

Given the following `config/mail.py` file:

```python
from masonite.environment import env

FROM_EMAIL = env("MAIL_FROM", "no-reply@masonite.com")

DRIVERS = {
    "default": env("MAIL_DRIVER", "terminal"),
    "smtp": {
        "host": env("MAIL_HOST"),
        "port": env("MAIL_PORT", "587"),
        "username": env("MAIL_USERNAME"),
        "password": env("MAIL_PASSWORD"),
        "from": FROM_EMAIL,
    }
}
```

- Accessing `mail` will return a dictionary with all the options.
- Accessing `mail.from_email` will return the `FROM_EMAIL` value
- Accessing `mail.drivers.smtp.port` will return the port value for smtp driver.


# Getting Value

To read a configuration value one can use the `Config` facade:

```python
from masonite.facades import Config

Config.get("mail.from_email")
# a default value can be provided
Config.get("database.mysql.port", 3306)
```

or the `config` helper:

```python
from masonite.configuration import config

config("mail.from_email")
# a default value can be provided
config("database.mysql.port", 3306)
```

# Setting Value

Setting configuration values is achieved through projet configuration files.


# Overriding Value

However, you can override on the fly a configuration value with the `Config` facade:

```python
Config.set("mail.from_email", "support@masoniteproject.com")
```

{% hint style="warning" %}
This should be done sparingly as this could have unexpected side effects depending at which time you override the configuration option.
{% endhint %}

This is mostly useful during tests, when you want to override a configuration option to test a specific behaviour:

```python
    def test_something(self):
        old_value = Config.get("mail.from_email")
        Config.set("mail.from_email", "support@masoniteproject.com")

        # test...

        Config.set("mail.from_email", old_value)
```

But if you simply want to have different configuration depending on the environment (development, testing or production) you should rely instead on [environment variables](../features/environments.md#) used to define configuration options.
