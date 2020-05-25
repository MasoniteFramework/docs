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

# Modifiers

In addition to the available columns you can use, you can also specify some modifers which will change the behavior of the column:

# Indexes

In addition to columns, you can also create indexes. Below are the available indexes you can create:

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
| .on_update('set null')  | Sets the ON UPDATE SET NULL property on the constraint.  |
| .on_update('cascade')  | Sets the ON UPDATE CASCADE property on the constraint.  |
| .on_delete('set null')  | Sets the ON DELETE SET NULL property on the constraint.  |
| .on_delete('cascade')  | Sets the ON DELETE CASCADE property on the constraint.  |

# Rolling Back

In addition to building up the migration, you should also build onto the `down` method which should reverse whatever was done in the `up` method. If you create a table in the up method, you should drop the table in the down method.

# Refreshing

Refreshing a database is simply rolling back all migrations and then migrating again. This "refreshes" your database.

You can refresh by running the command:

```
$ masonite-orm migrate:refresh
```

# Getting Migration Status

At any time you can get the migrations that have run or need to be ran:

```
$ masonite-orm migrate:status
```

# Changing Columns

**There currently is no "change" functionality, yet.
In order to change a column you currently will have to drop the column and then create a new one**

```python
class MigrationForUsersTable(Migration):
    def up(self):
        """
        Run the migrations.
        """
        with self.schema.table("users") as table:
            table.drop_column('email')

        with self.schema.table("users") as table:
            table.string('email').unique()


    def down(self):
        """
        Revert the migrations.
        """
        pass
```



