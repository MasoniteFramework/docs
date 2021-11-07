By default, your tests are are not ran in isolation from a database point of view. It means that your local database will be modified any time you run your tests and won't be rollbacked at the end of the tests.
While this behaviour might be fine in most case you can learn below how to configure your tests cases to reset the database after each test.

## Resetting The Database After Each Test

If you want to have a clean database for each test you must subclass the `TestCase` class with `DatabaseTransactions` class. Then all your tests will run inside a transaction so any data you create will only exist within the lifecycle of the test. Once the test completes, your database is rolled back to its previous state. This is a perfect way to prevent test data from clogging up your database.

```python
from masonite.tests import TestCase, DatabaseTransactions

class TestSomething(TestCase, DatabaseTransactions):

  connection = "testing"

  def test_can_create_user(self):
      User.create({"name": "john", "email": "john6", "password": "secret"})
```

Note that you can define the `connection` that will be used during testing. This will allow you to select a different database that will be used for testing. Here is a standard exemple of database configuration file that you can use.

```python
# config/database.py
DATABASES = {
    "default": "mysql",
    "mysql": {
        "host": "localhost",
        "driver": "mysql",
        "database": "app",
        "user": "root",
        "password": "",
        "port": 3306
    }
    "testing": {
        "driver": "sqlite",
        "database": "test_database.sqlite3",
    },
}
```

## Available Assertions

Masonite provides several database assertions that can be used during testing.

- [assertDatabaseCount](#assertdatabasecount)
- [assertDatabaseHas](#assertdatabasehas)
- [assertDatabaseMissing](#assertdatabasemissing)
- [assertDeleted](#assertdeleted)
- [assertSoftDeleted](#assertsoftdeleted)

### assertDatabaseCount

Assert that a table in the database contains the given number of records.

```python
self.assertDatabaseCount(table, count)
```

```python
  def test_can_create_user(self):
      User.create({"name": "john", "email": "john6", "password": "secret"})
      self.assertDatabaseCount("users", 1)
```

### assertDatabaseHas

Assert that a table in the database contains records matching the given query.

```python
self.assertDatabaseHas(table, query_dict)
```

```python
self.assertDatabaseCount("users", {"name": "John"})
```

### assertDatabaseMissing

Assert that a table in the database does not contain records matching the given query.

```python
self.assertDatabaseMissing(table, query_dict)
```

```python
self.assertDatabaseMissing("users", {"name": "Jack"})
```

### assertDeleted

Assert that the given model instance has been deleted from the database.

```python
user=User.find(1)
user.delete()
self.assertDeleted(user)
```

### assertSoftDeleted

Assert that the given model instance has been [soft deleted](https://orm.masoniteproject.com/models#soft-deleting) from the database.

```python
self.assertSoftDeleted(user)
```
