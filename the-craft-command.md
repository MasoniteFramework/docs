# The Craft Command

# Introduction

The craft command tool is a powerful developer tool that lets you quickly scaffold your project with models, controllers and views as well as condense nearly everything down to it’s simplest form via the craft namespace.

For example, In Django you may need to do something like:

```
$ django-admin startproject
$ python manage.py runserver
$ python manage.py migrate
$ git push heroku master
```

The craft tool condenses all commonly used commands into its own namespace

```
$ craft new
$ craft serve
$ craft migrate
$ craft deploy
```

Not all commands can be encompassed into craft commands out of the box but the most common ones are supported.

All scaffolding of Masonite can be done manually \(manually creating a controller and importing the `view` function for example\) but the craft command tool is used for speeding up development and cutting down on mundane development time.

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

This command will create a new controller under `app/http/controller`. By convention, all controllers should have an appended “Controller”. For example in order to make a dashboard controller, you should run `craft controller DashboardController` and not `craft controller Dashboard` although you can name your controllers however you’d like.

### Deployment

If you’d like to deploy your application to Heroku, Masonite comes with a command for that out of the box. You do not have to use this command and you do not have to deploy to Heroku. You may choose any deployment site but for quick development purposes, it might be convenient for you to quickly upload to Heroku to test a typical deployment.

```
$ craft deploy
```

Read the “Deployment” documentation for more information on deploying Masonite.

### Creating a New Project

If you’d like to start a new project, you can run:

```
$ craft new project_name
```

This will download a zip file over the internet and unzip it into your directory.

After you have created a new project, you will have a `requirements.txt` file with all of the projects dependencies. In addition to this file, you will also have a `.env-example` file which contains a boiler plate of a `.env` file. In order to install the dependencies, as well as copy the example environment file to a `.env` file, just run:

```
$ craft install
```

### Creating Migrations

All frameworks have a way to create migrations in order to manipulate database tables. Masonite uses a little bit of a different approach to migrations than other Python frameworks and makes the developer edit the migration file. More information on why can be found in the “Why not Django?” documentation. This is the command to make a migration for an existing table:

```
$ craft migration name_of_migration —-table users
```

If you are creating a migration for a table that does not exist yet which the migration will create it, you can pass the `--create` flag like so:

```
$ craft migration name_of_migration --create users
```

### Migrating

After your migrations have been created, edited, and are ready for migrating, we can now migrate them into the database. To migrate all of your immigrated migrations, just run:

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

Lastly, you can rollback just the last migration you tried migrating

```
$ craft migrate:rollback
```

### Models

If you'd like to create a model, you can run:

```
$ craft model ModelName
```

This will scaffold a model under \`app/ModelName\` and import everything needed.

If you need to create a model in a specific folder starting from the `app` folder, then just run:

```
$ craft model Models\ModelName
```

This will create a model in `app/Models/ModelName.py.`

### Creating a Service Provider

Service Providers is a really powerful feature of Masonite. If you'd like to create your own service provider, just run:

```
$ craft provider DashboardProvider
```

This will create a file at `app/providers/DashboardProvider.py`

### Creating Views

Views are simply html files located in `resources/templates` and can be created easily from running the command:

```
$ craft view blog
```

This command will create a template at `resources/templates/blog.html`

### Packages

You may create a PyPi package with an added `integrations.py` file which is specific to Masonite. You can learn more about packages by reading the "Creating Packages" documentation. To create a package boilerplate, just run:

```
$ craft package name_of_package
```

### Publishing

Packages that are built specifically for Masonite in mind will typically support publishing commands. Publishing commands are a way that packages can scaffold and integrate into Masonite. Publishing commands can allow third parties to: create or append to configuration files, create controllers, create routes and other integrations. Read more about publishing by reading our "Publishing Packages" documentation. To publish a package just run:

```
$ craft publish name_of_package
```

### Running the WSGI Server

You can run the WSGI server by simply running:

```
$ craft serve
```

### Encryption

Masonite comes with a way to encrypt data and by default, encrypts all cookies set by the framework. Masonite uses a `key` to encrypt and decrypt data. Read the "Encryption" documentation for more information on encryption.

To generate a secret `key`, we can run:

```
$ craft key
```

This will generate a 32 bit string which you can paste into your `.env` file under the `KEY` setting.

Great! You are now a master at the craft command line tool.