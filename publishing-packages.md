# Publishing Packages

# Introduction

Publishing packages are a great way for third party packages to integrate into Masonite. They are extremely easy to setup even on existing pip packages. Publishing packages allows any package to create configuration files, routes, controllers, and other integrations that make developing with your packages amazing. You can read about this in the [Creating Packages](/creating-packages.md) documentation to learn more about how you can integrate your existing packages or future packages with Masonite.

#### NOTE: Virtual Environment {#note-virtual-environment}

**If you are in a virtual environment,**`craft publish`**will not have access to your virtual environment dependencies. In order to fix this, we can add our site packages to our**`config/packages.py`**config file**

If you attempt to publish your package without your virtual environment's site\_packages file being inside `config/packages.py`, you will encounter a `ModuleNotFound` error that says it cannot find the integrations file located with the package you are trying to install. This is because Masonite does not have knowledge of your virtual environments dependencies, only your system dependencies.

If you are in a virtual environment then go to your`config/packages.py`file and add your virtual environments site\_packages folder to the`SITE_PACKAGES`list. Your`SITE_PACKAGES`list may look something like:

```
SITE_PACKAGES = [
    'venv/lib/python3.6/site-packages'
]
```

This will allow`craft publish`to find our dependencies installed on our virtual environment. Read the [Publishing Packages](/publishing-packages.md) documentation for more information.

Once done, all future packages that you pip install will be available through the publish command. This configuration should be done as soon as your virtual environment is created so you don't encounter any errors while trying to publish packages.