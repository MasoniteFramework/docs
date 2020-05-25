# Schema and Migrations

Migrations are used to build and modify your database tables. This is done through use of migration files and the `Schema` class. Migration files are really just wrappers around the `Schema` class as well as a way for Masonite to manage which migrations have run and which ones have not.

# Creating Migrations

Creating migrations are easy with the migration commands. To create one simply run:

```
$ masonite-orm migration migration_for_users_table
```

This will create a migration file for you and put it in the `databases/migrations` directory. 

If you want to create a starter migration, that is a migration with some boilerplate of what you are planning to do, you can use the `--table` and `--create` flag:

```
$ masonite-orm migration migration_for_users_table --create users
```

This will setup a migration for you with some boiler plate on creating a new table

```
$ masonite-orm migration migration_for_users_table --table users
```

This will setup a migration for you for boiler plate on modifying an existing table.

# Building Migrations

To start building up your migration, simply modify the `up` method and start adding any of the available methods below to your migration.

A simple example would look like this for a new table:

```python
class MigrationForUsersTable(Migration):
    def up(self):
        """
        Run the migrations.
        """
        with self.schema.create("users") as table:
            table.increments('id')
            table.string('username')
            table.string('email').unique()
            table.string('password')
            table.bool('is_admin')
            table.integer('age')
            
            table.timestamps()

    def down(self):
        """
        Revert the migrations.
        """
        self.schema.drop("users")
```

## Available Methods

|Command | Description | 
|---|---|
| table.string()  |   |
| table.integer() |   |
| table.increments()  |   |
| table.big_increments()  |   |
| table.binary()  |   |
| table.boolean()  |   |
| table.char()  |   |
| table.date()  |   |
| table.datetime()  |   |
| table.timestamp()  |   |
| table.timestamps()  |   |
| table.decimal()  |   |
| table.double()  |   |
| table.enum()  |   |
| table.text()  |   |
| table.unsigned_integer()  |   |
| table.unsigned()  |   |
| table.text()  |   |
| table.text()  |   |
| table.text()  |   |

# Modifiers

In addition to the available columns you can use, you can also specify some modifers which will change the behavior of the column:

|Command | Description | 
|---|---|
| .nullable()  |   |
| .unique() |   |
| .after() |   |
| .unsigned() |   |
| .use_current() |   |

# Indexes

In addition to columns, you can also create indexes. Below are the available indexes you can create:

|Command | Description | 
|---|---|
| table.primary()  |   |
| table.unique() |   |
| table.index() |   |

# Foreign Keys

If you want to create a foreign key you can do so simply as well:

```python
table.foreign('local_column').references('other_column').on('other_table')
```

And optionally specify an `on_delete` or `on_update` method:

```python
table.foreign('local_column').references('other_column').on('other_table').on_update('set null')
```

You can use these options:

|Command | Description | 
|---|---|
| .on_update('set null')  |   |
| .on_update('cascade')  |   |
| .on_delete('set null')  |   |
| .on_delete('cascade')  |   |

# Rolling Back

## Dropping tables

## Dropping columns

## Dropping indexes

# Refreshing

# Getting Migration Status

# Changing Columns

