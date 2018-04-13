# Database Migrations

## Introduction

Database migrations in Masonite is very different than other Python frameworks. Other Python frameworks create migrations based on a model which historically uses Data Mapper type ORM's. Because Masonite uses an Active Record ORM by default, Migrations are completely separated from models. This is great as it allows a seamless switch of ORM's without interfering with migrations. In addition to creating this separation of migrations and models, it makes managing the relationship between models and tables extremely basic with very little magic which leads to faster debugging as well as fewer migration issues.

In this documentation, we'll talk about how to make migrations with Masonite.

## Getting Started

Because models and migrations are separated, we never have to touch our model in order to make alterations to our database tables. In order to make a migration we can run a craft command:

```text
$ craft migration name_of_migration_here --table dashboard
```

This command will create a migration for an existing table. A migration on an existing table will migrate into the database in a certain way so it's important to specify the `--table` flag in the command.

In order to create a migration file for a new table that doesn't yet exist \(but will after the migration\) you can instead use the `--create` flag like so:

```text
$ craft migration name_of_migration_here --create dashboard
```

This will create a migration that will create a table, as well as migrate the columns you specify.

### Migrating Columns

Inside the migration file you will see an `up()` method and a `down()` method. We are only interested in the `up()` method. This method specifies what to do when the migration file is migrated. The `down()` method is what is executed when the migration is rolled back. Lets walk through creating a blog migration.

We can use [Orators Schema Builder](https://orator-orm.com/docs/0.9/schema_builder.html) in order to build our migration file. First lets run a migration craft command to create a blog table:

```text
$ craft migration create_blogs_table --create blogs
```

This will create a migration file located in `databases/migrations` . Lets open this file and add some columns.

After we open it we should see something an up\(\) method that looks like this:

```python
def up(self):
        """
        Run the migrations.
        """
        with self.schema.create('blogs') as table:
            table.increments('id')
            table.timestamps()
```

Inside our with statement we can start adding columns.

Lets go ahead and add some columns that can be used for a blog table.

```python
def up(self):
        """
        Run the migrations.
        """
        with self.schema.create('blogs') as table:
            table.increments('id')
            table.string('title')
            table.text('body')
            table.integer('active')
            table.integer('user_id').unsigned()
            table.foreign('user_id').references('id').on('users')
            table.timestamps()
```

Ok let's go ahead and break down what we just created.

Notice that we are using a context processor which is our schema builder. All we have to worry about is whats inside it. Notice that we have a `table` object that has a few methods that are related to columns. Most of these columns are pretty obvious and you can read about different [Orator Schema Columns](https://orator-orm.com/docs/0.9/schema_builder.html#adding-columns) you can use. We'll mention the foreign key here though.

### Foreign Keys

So adding columns is really straight forward and Orator has some great documentation on their website. In order to add a foreign key, we'll need an unsigned integer column which we specified above called:

```python
table.integer('user_id').unsigned()
```

This will set up our column index to be ready for a foreign key. We can easily specify a foreign key by then typing

```text
table.foreign('user_id').references('id').on('users')
```

What this does is sets a foreign key on the `user_id` column which references the `id` column on the `users` table. That's it! It's that easy and expressive to set up a foreign key.

Check the [Orator documentation](https://orator-orm.com/docs/0.9/schema_builder.html#adding-columns) for more information on creating a migration file.

### Changing Columns

There are two types of columns that we will need to change over the course of developing our application. Changing columns is extremely simple. If you're using MySQL 5.6.6 and below, see the caveat below.

To change a column, we can just use the `.change()` method on it. Since we need to create a new migration to do this, we can do something like:

```python
$ craft migration change_default_status --table dashboard
```

and then simply create a new migration but use the `.change()` method to let Masonite you want to change an existing column instead of adding a new one:

```python
table.integer('status').nullable().default(0).change()
```

When we run `craft migrate` it will change the column instead of adding a new one.

### Changing Foreign Keys Prior to MySQL 5.6.6

Because of the constraints that foreign keys put on columns prior to MySQL 5.6.6, it's not as straight forward as appending a `.change()` to the foreign key column. We must first:

* drop the foreign key relationship
* change the column
* recreate the foreign key

We can do this simply like so:

```python
table.drop_foreign('posts_user_id_foreign')
table.rename_column('user_id', 'author_id')
table.foreign('author_id').references('id').on('users')
```

