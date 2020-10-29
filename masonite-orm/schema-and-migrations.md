# Schema and Migrations

## Schema and Migrations

Migrations are used to build and modify your database tables. This is done through use of migration files and the `Schema` class. Migration files are really just wrappers around the `Schema` class as well as a way for Masonite to manage which migrations have run and which ones have not.

## Creating Migrations

Creating migrations are easy with the migration commands. To create one simply run:

```text
$ masonite-orm migration migration_for_users_table
```

This will create a migration file for you and put it in the `databases/migrations` directory.

If you want to create a starter migration, that is a migration with some boilerplate of what you are planning to do, you can use the `--table` and `--create` flag:

```text
$ masonite-orm migration migration_for_users_table --create users
```

This will setup a migration for you with some boiler plate on creating a new table

```text
$ masonite-orm migration migration_for_users_table --table users
```

This will setup a migration for you for boiler plate on modifying an existing table.

## Building Migrations

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

### Available Methods

| Command | Description |
| :--- | :--- |
| `table.string()` | The varchar version of the table. Can optional pass in a length `table.string('name', length=181)` |
| `table.integer()` | The INT version of the database. Can also specify a length `table.integer('age', length=5)` |
| `table.increments()` | The auto incrementing version of the table. An unsigned non nullable auto incrementing integer. |
| `table.big_increments()` | An unsigned non nullable auto incrementing big integer. Use this if you expect the rows in a table to be very large |
| `table.binary()` | BINARY equivalent column. Sometimes is text field on unsupported databases. |
| `table.boolean()` | BOOLEAN equivalent column. |
| `table.char()` | CHAR equivalent column. |
| `table.date()` | DATE equivalent column. |
| `table.datetime()` | DATETIME equivalent column. |
| `table.timestamp()` | TIMESTAMP equivalent column. |
| `table.timestamps()` | Creates `created_at` and `updated_at` columns on the table with the `timestamp` column and defaults to the current time. |
| `table.decimal()` | DECIMAL equivalent column. Can also specify the length and decimal position. `table.decimal('salary', 17, 6)` |
| `table.double()` | DOUBLE equivalent column. Can also specify a float length `table.double('salary', 17,6)` |
| `table.enum()` | ENUM equivalent column. You can also specify available options as a list. `table.enum('flavor', ['chocolate', 'vanilla'])`. Sometimes defaults to a TEXT field with a constraint on unsupported databases. |
| `table.text()` | TEXT equivalent column. |
| `table.unsigned_integer()` | UNSIGNED INT equivalent column. |
| `table.unsigned()` | Alias for `unsigned_integer` |

## Modifiers

In addition to the available columns you can use, you can also specify some modifers which will change the behavior of the column:

| Command | Description |
| :--- | :--- |
| .nullable\(\) | Allows NULL values to be inserted into the column. |
| .unique\(\) | Forces all values in the column to be unique. |
| .after\(\) | Adds the column after another column in the table. Can be used like `table.string('is_admin').after('email')`. |
| .unsigned\(\) | Makes the column unsigned. Used with the `table.integer('age').unsigned()` column. |
| .use\_current\(\) | Makes the column use the `CURRENT_TIMESTAMP` modifer. |

## Indexes

In addition to columns, you can also create indexes. Below are the available indexes you can create:

| Command | Description |
| :--- | :--- |
| `table.primary()` | Make the column use the PRIMARY KEY modifer. |
| `table.unique()` | Makes a unique index. Can pass in a column `table.unique('email')` or list of columns `table.unique(['email', 'phone_number'])`. |
| `table.index()` | Creates an index on the column. `table.index('email')` |

The default primary key is often set to an auto-incrementing integer, but you can [use UUID instead](models.md#changing-primary-key-to-use-uuid).

## Foreign Keys

If you want to create a foreign key you can do so simply as well:

```python
table.foreign('local_column').references('other_column').on('other_table')
```

And optionally specify an `on_delete` or `on_update` method:

```python
table.foreign('local_column').references('other_column').on('other_table').on_update('set null')
```

You can use these options:

| Command | Description |
| :--- | :--- |
| .on\_update\('set null'\) | Sets the ON UPDATE SET NULL property on the constraint. |
| .on\_update\('cascade'\) | Sets the ON UPDATE CASCADE property on the constraint. |
| .on\_delete\('set null'\) | Sets the ON DELETE SET NULL property on the constraint. |
| .on\_delete\('cascade'\) | Sets the ON DELETE CASCADE property on the constraint. |

## Rolling Back

In addition to building up the migration, you should also build onto the `down` method which should reverse whatever was done in the `up` method. If you create a table in the up method, you should drop the table in the down method.

| Command | Description |
| :--- | :--- |
| `table.drop_table()` | DROP TABLE equivalent statement. |
| `table.drop_table_if_exists()` | DROP TABLE IF EXISTS equivalent statement. |
| `table.drop_column()` | DROP COLUMN equivalent statement. |
| `table.drop_index()` | Drops the constraint. Must pass in the name of the constraint. `drop_index('email_index')` |
| `table.drop_unique()` | Drops the uniqueness constraint. Must pass in the name of the constraint. `table.drop_unique('users_email_unique')` |
| `table.drop_foreign()` | Drops the foreign key. Must specify the index name. `table.drop_foreign('users_article_id_foreign')` |
| `table.drop_primary()` | Drops the primary key constraint. Must pass in the constraint name `table.drop_foreign('users_id_primary')` |

## Refreshing

Refreshing a database is simply rolling back all migrations and then migrating again. This "refreshes" your database.

You can refresh by running the command:

```text
$ masonite-orm migrate:refresh
```

## Getting Migration Status

At any time you can get the migrations that have run or need to be ran:

```text
$ masonite-orm migrate:status
```

## Seeing SQL Dumps

If you would like to see just the SQL that would run instead of running the actual migrations, you can specify the `-s` flag (short for `--show`). This works on the migrate and migrate:rollback commands.

```
python craft migrate -s
```

## Changing Columns

If you would like to change a column you should simply specify the new column and then specify a `.change()` method on it.

Here is an example of changing an email field to a nullable field:

```python
class MigrationForUsersTable(Migration):
    def up(self):
        """
        Run the migrations.
        """
        with self.schema.table("users") as table:
            table.string('email').nullable().change()

        with self.schema.table("users") as table:
            table.string('email').unique()


    def down(self):
        """
        Revert the migrations.
        """
        pass
```

