# The Craft Command

# Introduction

The craft command tool is a powerful developer tool that lets you quickly scaffold your project with models, controllers and views as well as condense nearly everything down to it’s simplest form via the craft namespace.

For example, In Django you may need to do something like:

```
$ django-admin startproject
$ python manage.py runserver
$ python manage.py migrate
```

The craft tool condenses all commonly used commands into its own namespace

```
$ craft new
$ craft serve
$ craft migrate
```

All scaffolding of Masonite can be done manually (manually creating a controller and importing the `view` function for example) but the craft command tool is used for speeding up development and cutting down on mundane development time.

## Commands

The possible commands for craft include:

### Creating an Authentication System

To create an authentication system with a login, register and a dashboard system, just run:

```
 $ craft auth
```

This command will create several new templates, controllers and routes so you don’t need to create an authentication system from scratch, although you can. If you need a custom authentication system, this command will scaffold the project for you so you can go into these new controllers and change them how you see fit.

These new controllers are not apart of the framework itself but now apart of your project. Do not look at editing these controllers as editing the framework source code.

### Creating Controllers

If you wish to scaffold a controller, just run:

```
$ craft controller
```

This command will create a new controller under `app/http/controller`. By convention, all controllers should have an appended “Controller”. For example in order to make a dashboard controller, you should run `craft controller DashboardController` and not `craft controller Dashboard` although you can name your controllers however you like.

### Creating a New Project

If you’d like to start a new project, you can run:

```
$ craft new project_name
```

This will download a zip file of the `MasoniteFramework/masonite` repository and unzip it into your current working directory. This command will default to the latest release of the repo.

You may also specify some options. The `--version` option will create a new project depending on the releases from the `MasoniteFramework/masonite` repository.

```
$ craft new project_name --version 1.3.0
```

Or you can specify the branch you would like to create a new project with:

```
$ craft new project_name --branch develop
```

After you have created a new project, you will have a `requirements.txt` file with all of the projects dependencies. In addition to this file, you will also have a `.env-example` file which contains a boiler plate of a `.env` file. In order to install the dependencies, as well as copy the example environment file to a `.env` file, just run:

```
$ craft install
```

The `craft install` command will also run `craft key --store` as well which generates a secret key and places it in the `.env` file.

### Creating Migrations

All frameworks have a way to create migrations in order to manipulate database tables. Masonite uses a little bit of a different approach to migrations than other Python frameworks and makes the developer edit the migration file. This is the command to make a migration for an existing table:

```
$ craft migration name_of_migration —-table users
```

If you are creating a migration for a table that does not exist yet which the migration will create it, you can pass the `--create` flag like so:

```
$ craft migration name_of_migration --create users
```

These two flags will create slightly different types of migrations.

### Migrating

After your migrations have been created, edited, and are ready for migrating, we can now migrate them into the database. To migrate all of your unmigrated migrations, just run:

```
$ craft migrate
```

You can also refresh and rollback all of your migrations and remigrate them. **This will basically rebuild your entire database.**

```
$ craft migrate:refresh
```

You can also rollback all migrations without remigrating

```
$ craft migrate:reset
```

Lastly, you can rollback just the last set of migrations you tried migrating

```
$ craft migrate:rollback
```

### Models

If you'd like to create a model, you can run:

```
$ craft model ModelName
```

This will scaffold a model under `app/ModelName` and import everything needed.

If you need to create a model in a specific folder starting from the `app` folder, then just run:

```
$ craft model Models/ModelName
```

This will create a model in `app/Models/ModelName.py.`

### Creating a Service Provider

Service Providers are a really powerful feature of Masonite. If you'd like to create your own service provider, just run:

```
$ craft provider DashboardProvider
```

This will create a file at `app/providers/DashboardProvider.py`

Read more about Service Providers under the [Service Provider](/service-container/service-providers.md) documentation.

### Creating a Job

Jobs are used for Masonite's queue systems. You can create these `Queueable` classes and they will be able to be loaded into different queues. To create a job, run:

```
$ craft job JobName
```

This will create a job inside the `app/jobs` directory.
  

### Creating Views

Views are simply html files located in `resources/templates` and can be created easily from running the command:

```
$ craft view blog
```

This command will create a template at `resources/templates/blog.html`

You can also create a view deeper inside the `resources/templates` directory.

```
$ craft view auth/home
```

This will create a view under `resources/templates/auth/home.html` but keep in mind that it will not create the directory for you. If the `auth` directory does not exist, this command will fail.

### Creating Jobs

Jobs are designed to be loaded into queues. We can take time consuming tasks and throw them inside of a Job. We can then use this Job to push to a queue to speed up the performance of our application and prevent bottlenecks and slowdowns.

```
$ craft job SendWelcomeEmail
```

Jobs will be put inside the `app/jobs` directory. See the [Queues and Jobs](/queues.md) documentation for more information.

### Packages

You may create a PyPi package with an added `integrations.py` file which is specific to Masonite. You can learn more about packages by reading the [Creating Packages](/creating-packages.md) documentation. To create a package boilerplate, just run:

```
$ craft package name_of_package
```

### Publishing

Packages that are built specifically for Masonite in mind will typically support publishing commands. Publishing commands are a way that packages can scaffold and integrate into Masonite. Publishing commands can allow third parties to: create or append to configuration files, create controllers, create routes and other integrations. Read more about publishing by reading our [Publishing Packages](/publishing-packages.md) documentation. To publish a package just run:

```
$ craft publish name_of_package
```

### Running the WSGI Server

You can run the WSGI server by simply running:

```
$ craft serve
```

### Encryption

Masonite comes with a way to encrypt data and by default, encrypts all cookies set by the framework. Masonite uses a `key` to encrypt and decrypt data. Read the [Encryption](/security/encryption.md) documentation for more information on encryption.

To generate a secret `key`, we can run:

```
$ craft key
```

This will generate a 32 bit string which you can paste into your `.env` file under the `KEY` setting.

You may also pass the `--store` flag which will automatically add the key to your `.env` file:

```
$ craft key --store
```

This command is ran whenever you run `craft install`

Great! You are now a master at the craft command line tool.

