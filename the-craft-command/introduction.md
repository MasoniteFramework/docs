# Introduction

## Introduction

The craft command tool is a powerful developer tool that lets you quickly scaffold your project with models, controllers, views, commands, providers, etc. which will condense nearly everything down to it’s simplest form via the craft namespace. No more redundancy in your development time creating boilerplate code. Masonite condenses all common development tasks into a single namespace.

For example, In Django you may need to do something like:

```text
$ django-admin startproject
$ python manage.py runserver
$ python manage.py migrate
```

The craft tool condenses all commonly used commands into its own namespace

```text
$ craft new
$ craft serve
$ craft migrate
```

All scaffolding of Masonite can be done manually \(manually creating a controller and importing the `view` function for example\) but the craft command tool is used for speeding up development and cutting down on mundane development time.

## Commands

When craft is used outside of masonite directory, it will only show a few commands such as the `new` and `install` commands. Other commands such as commands for creating controllers or models are loaded in from the Masonite project itself.

{% hint style="info" %}
Many commands are loaded into the framework itself and fetched when craft is ran in a Masonite project directory. This allows version specific Masonite commands to be efficiently handled on each subsequent version as well as third party commands to be loaded in which expands craft itself.
{% endhint %}

The possible commands for craft include:

### Tinker Command

You can "tinker" around with Masonite by running:

```text
$ craft tinker
```

This command will start a Python shell but also imports the container by default. So we can call:

```bash
Type `exit()` to exit.
>>> app
<masonite.app.App object at 0x10cfb8d30>
>>> app.make('Request')
<masonite.request.Request object at 0x10d03c8d0>
>>> app.collect("Mail*Driver")
{'MailSmtpDriver': <class 'masonite.drivers.MailSmtpDriver.MailSmtpDriver'>,
'MailMailgunDriver': <class 'masonite.drivers.MailMailgunDriver.MailMailgunDriver'>
}
>>> exit()
```

And play around the container. This is a useful debugging tool to verify that objects are loaded into the container if there are any issues.

### Show Routes Command

Another useful command is the show:routes command which will display a table of available routes that can be hit:

```text
$ craft show:routes
```

This will display a table that looks like:

```text
========  =======  =======  ========  ============
Method    Path     Name     Domain    Middleware
========  =======  =======  ========  ============
GET       /        welcome
GET       /home    home
GET       /user 
POST      /create  user   
========  =======  =======  ========  ============
```

### Application Information

If you are trying to debug your application or need help in the Slack channel, it might be beneficial to see some useful information information about your system and environment. In this case we have a simple command:

```text
$ craft info
```

This will give some small details about the current system which could be useful to someone trying to help you. Running the commnad will give you something like this:

```text
Environment Information
-------------------------  ------------------
System Information         MacOS x86_64 64bit
System Memory              8 GB
Python Version             CPython 3.6.5
Virtual Environment        ✓
Masonite Version           2.0.6
Craft Version              2.0.7
APP_ENV                    local
APP_DEBUG                  True
```

Feel free to contribute any additional information you think is necessary to the command in the core repository.

### Creating an Authentication System

To create an authentication system with a login, register and a dashboard system, just run:

```text
 $ craft auth
```

This command will create several new templates, controllers and routes so you don’t need to create an authentication system from scratch, although you can. If you need a custom authentication system, this command will scaffold the project for you so you can go into these new controllers and change them how you see fit.

These new controllers are not apart of the framework itself but now apart of your project. Do not look at editing these controllers as editing the framework source code.

### Creating Controllers

If you wish to scaffold a controller, just run:

```text
$ craft controller Dashboard
```

This command will create a new controller under `app/http/controllers/DashboardController.py`. By convention, all controllers should have an appended “Controller” and therefore Masonite will append "Controller" to the controller created. 

You can create a controller without appending "Controller" to the end by running:

```text
$ craft controller Dashboard -e
```

This will create a app/http/controllers/Dashboard.py file with a Dashboard controller. Notice that "Controller" is not appended.

{% hint style="info" %}
`-e` is short for `--exact`. Either flag will work.
{% endhint %}

You may also create resource controllers which include standard resource actions such as show, create, update, etc:

```text
$ craft controller Dashboard -r
```

{% hint style="info" %}
`-r` is short for `--resource`. Either flag will work.
{% endhint %}

You can also obviously combine them:

```text
$ craft controller Dashboard -r -e
```

### Creating a New Project

If you’d like to start a new project, you can run:

```text
$ craft new project_name
```

This will download a zip file of the `MasoniteFramework/masonite` repository and unzip it into your current working directory. This command will default to the latest release of the repo.

You may also specify some options. The `--version` option will create a new project depending on the releases from the `MasoniteFramework/masonite` repository.

```text
$ craft new project_name --version 1.3.0
```

Or you can specify the branch you would like to create a new project with:

```text
$ craft new project_name --branch develop
```

After you have created a new project, you will have a `requirements.txt` file with all of the projects dependencies. In addition to this file, you will also have a `.env-example` file which contains a boiler plate of a `.env` file. In order to install the dependencies, as well as copy the example environment file to a `.env` file, just run:

```text
$ craft install
```

The `craft install` command will also run `craft key --store` as well which generates a secret key and places it in the `.env` file.

### Creating Migrations

All frameworks have a way to create migrations in order to manipulate database tables. Masonite uses a little bit of a different approach to migrations than other Python frameworks and makes the developer edit the migration file. This is the command to make a migration for an existing table:

```text
$ craft migration name_of_migration --table users
```

If you are creating a migration for a table that does not exist yet which the migration will create it, you can pass the `--create` flag like so:

```text
$ craft migration name_of_migration --create users
```

These two flags will create slightly different types of migrations.

### Migrating

After your migrations have been created, edited, and are ready for migrating, we can now migrate them into the database. To migrate all of your unmigrated migrations, just run:

```text
$ craft migrate
```

### Rolling Back and Rebuilding Migrations

You can also refresh and rollback all of your migrations and remigrate them.

{% hint style="danger" %}
This will essentially rebuild your entire database.
{% endhint %}

```text
$ craft migrate:refresh
```

You can also rollback all migrations without remigrating

```text
$ craft migrate:reset
```

Lastly, you can rollback just the last set of migrations you tried migrating

```text
$ craft migrate:rollback
```

### Models

If you'd like to create a model, you can run:

```text
$ craft model ModelName
```

This will scaffold a model under `app/ModelName` and import everything needed.

If you need to create a model in a specific folder starting from the `app` folder, then just run:

```text
$ craft model Models/ModelName
```

This will create a model in `app/Models/ModelName.py.`

### Creating a Service Provider

Service Providers are a really powerful feature of Masonite. If you'd like to create your own service provider, just run:

```text
$ craft provider DashboardProvider
```

This will create a file at `app/providers/DashboardProvider.py`

Read more about Service Providers under the [Service Provider](../architectural-concepts/service-providers.md) documentation.

### Creating Views

Views are simply html files located in `resources/templates` and can be created easily from running the command:

```text
$ craft view blog
```

This command will create a template at `resources/templates/blog.html`

You can also create a view deeper inside the `resources/templates` directory.

```text
$ craft view auth/home
```

This will create a view under `resources/templates/auth/home.html` but keep in mind that it will not create the directory for you. If the `auth` directory does not exist, this command will fail.

### Creating Jobs

Jobs are designed to be loaded into queues. We can take time consuming tasks and throw them inside of a Job. We can then use this Job to push to a queue to speed up the performance of our application and prevent bottlenecks and slowdowns.

```text
$ craft job SendWelcomeEmail
```

Jobs will be put inside the `app/jobs` directory. See the [Queues and Jobs](../useful-features/queues-and-jobs.md) documentation for more information.

### Packages

You may create a PyPi package with an added `integrations.py` file which is specific to Masonite. You can learn more about packages by reading the [Creating Packages](../advanced/creating-packages.md) documentation. To create a package boilerplate, just run:

```text
$ craft package name_of_package
```

### Creating Commands

You can scaffold out basic command boilerplate:

```text
$ craft command HelloCommand
```

This will create a `app/commands/HelloCommand.py` file with the `HelloCommand` class.

### Running the WSGI Server

You can run the WSGI server by simply running:

```text
$ craft serve
```

You will likely only use that command in production. For development, you can make the server autoreload after you save a python file, similiar to how Django does.

```text
$ craft serve --reload
```

or the shorthand

```text
$ craft serve -r
```

### Encryption

Masonite comes with a way to encrypt data and by default, encrypts all cookies set by the framework. Masonite uses a `key` to encrypt and decrypt data. Read the [Encryption](../security/encryption.md) documentation for more information on encryption.

To generate a secret `key`, we can run:

```text
$ craft key
```

This will generate a 32 bit string which you can paste into your `.env` file under the `KEY` setting.

You may also pass the `--store` flag which will automatically add the key to your `.env` file:

```text
$ craft key --store
```

This command is ran whenever you run `craft install`

Great! You are now a master at the craft command line tool.

