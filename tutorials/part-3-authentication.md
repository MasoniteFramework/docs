---
description: >-
  In this part we will talk about how to setup up an authentication system for
  our blog.
---

# Part 3 - Authentication

## Getting Started

Most applications will require some form of authentication. Masonite comes with a craft command to scaffold out an authentication system for you. This should typically be ran on fresh installation of Masonite since it will create controllers routes and views for you.

For our blog, we will need to setup some a registration form so we can get new users to start posting to our blog. We can create an authentication system by running the craft command:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft auth
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We should get a success message saying that some new assets were created. You can check your controllers folder and you should see a few new controllers there that should handle registrations.

## Database Setup

In order to register these users, we will need a database. Hopefully you already have some kind of local database setup like MySQL or Postgres. You can open up your MySQL client and create a database. You currently cannot create a database directly through Masonite.

Create a database and name it whatever you like. For the purposes of this tutorial, we can name it "blog"

Once that is done we just need to change a few environment variables so Masonite can connect to the database. These environment variable can be found in the `.env` file in the root of the project. Open that file up and you should see a few lines that look like:

{% code-tabs %}
{% code-tabs-item title=".env" %}
```text
DB_DRIVER=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=masonite
DB_USERNAME=root
DB_PASSWORD=root
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Go ahead and change those setting to your connection settings. The `DB_DRIVER` constant takes 3 values: `mysql`, `postgres` and `sqlite`.

{% hint style="warning" %}
Since it has a simpler configuration, if you are using `sqlite`, you will need to edit `config/database.py` to remove the `host`, `user`, and `password` keys.
{% endhint %}

## Migrating

Once you have set the correct credentials, we can go ahead and migrate the database. Out of the box, Masonite has a migration for a users table which will be the foundation of our user. You can edit this user migration before migrating but the default configuration will suit most needs just fine and you can always add column at a later date.

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft migrate
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create our users table for us along with a migrations table to keep track of any migrations we add later.

## Creating Users

Now that we have the authentication and the migrations all migrated in, let's create our first user. Remember that we ran craft auth so we have a few new templates and controllers.

Go ahead and run the server:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft serve
```
{% endcode-tabs-item %}
{% endcode-tabs %}

and head over to `localhost:8000/register` and fill out the form. You can use whatever name and email you like but for this purpose we will use:

```text
Username: demo
Email: demo@email.com
Password: password
```

Once that's done we can move on to the next part.

