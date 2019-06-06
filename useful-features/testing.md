# Testing

## Testing

### Introduction

Masonite testing is very simple. You can test very complex parts of your code with ease by just extending your class with a Masonite unit test class.

Although Masonite uses `pytest` to run tests, Masonite's test suite is based on `unittest`. SO you will use `unittest` syntax but run the tests with Pytest.

### Configuration

First, create a new test class in a testing directory. There is a craft command you can run to create tests for you so just run:

```text
$ craft test User
```

This will create a user test for us which we can work on. You can drag this test in any subdirectory you like.

This command will create a basic test like the one below:

{% code-tabs %}
{% code-tabs-item title="tests/test\\user.py" %}
```python
"""Example Testcase."""

from masonite.testing import TestCase


class TestUser(TestCase):

    transactions = True

    def setUp(self):
        super().setUp()

    def setUpFactories(self):
        pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

That's it! You're ready to start testing. Read on to learn how to start building your test cases.

## Calling Routes

We have a few options for testing our routes.

### Testing If a Route Exists:

To check if a route exists, we can simple use either get or post:

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_route_exists(self):
    self.assertTrue(self.get('/testing'))
    self.assertTrue(self.post('/testing'))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Testing If Route Has The Correct Name

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_route_has_the_correct_name(self):
    self.assertTrue(self.get('/testing').is_named('testing.route'))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Testing If A Route Has The Correct Middleware

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_route_has_route_middleware(self):
    assert self.get('/testing').has_middleware('auth', 'owner')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Testing If A Route Contains A String

This can be used to see if the template returned a specific value

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_view_contains(self):
    assert self.get('/login').contains('Login Here')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Checking 200 Status Code

You can easily check if the response is ok by using the `ok` method:

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_view_is_ok(self):
    assert self.get('/testing').ok()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Getting Output

You can get the output by using the capture output easily by calling the `captureOutput` method on your unit test:

```python
def test_get_output(self):
    with self.captureOutput() as o:
        print('hello world!')

    self.assertEqual(o, 'hello world!')
```

## Testing the Database

### Databases

By default, to prevent messing with running databases, database test cases are set to only run on the `sqlite` database. You can disable this by setting the `sqlite` attribute to `False`.

```python
from masonite.testing import TestCase

class TestUser(TestCase):

    """Start and rollback transactions for this test
    """
    transactions = True
    sqlite = False

    def setUp(self):
        super().setUp()
```

This will allow you to use whatever database driver you need.

### Transactions and Refreshing

By default, all your tests will run inside a transaction so any data you create will only exist within the lifecycle of the test. Once the test completes, your database is rolled back to its previous state. This is a perfect way to prevent test data from clogging up your database.

Although this is good for most use cases, you may want to actually migrate and refresh the entire migration stack. In this case you can set the `refreshes_database` attribute to `True`.

```python
from masonite.testing import TestCase

class TestUser(TestCase):

    """Start and rollback transactions for this test
    """
    transactions = False
    refreshes_database = True

    def setUp(self):
        super().setUp()
```

Now this will migrate and refresh the database.

{% hint style="danger" %}
Beware that this will destroy any database information you have.
{% endhint %}

### Factories

Factories are simply ways to seed some dummy data into your database. You can create a factory by making a method that accepts a faker argument and using that to seed data.

Masonite has a convenient method you can use that will run once the test first boots up called `setUpFactories`. **This will run once and only once and not between every test.**

Let's create the factory as well as use the setUpFactories method to run them now.

Below is how to create 100 users:

```python
from masonite.testing import TestCase
from app.User import User

class TestUser(TestCase):

    """Start and rollback transactions for this test
    """
    transactions = True

    def setUp(self):
        super().setUp()

    def setUpFactories(self):
        self.make(User, self.user_factory, 100)

    def user_factory(self, faker):
        return {
            'name': faker.name(),
            'email': faker.email(),
            'password': '$2b$12$WMgb5Re1NqUr.uSRfQmPQeeGWudk/8/aNbVMpD1dR.Et83vfL8WAu',  
            # == 'secret'
        }

    def test_creates_users(self):
        pass
```

You don't need to build factories though. This can be used to simply create new records:

```python
from masonite.testing import TestCase
from app.User import User

class TestUser(TestCase):

    """Start and rollback transactions for this test
    """
    transactions = True

    def setUp(self):
        super().setUp()

    def setUpFactories(self):
        User.create({
            'name': 'Joe',
            'email': 'user@example.com',
            'password': '$2b$12$WMgb5Re1NqUr.uSRfQmPQeeGWudk/8/aNbVMpD1dR.Et83vfL8WAu',  # == 'secret'
        })

    def test_creates_users(self):
        pass
```

### Users

We can load users into the route and check if they can view the route. This is good to see if your middleware is acting good against various users. This can be done with the `acting_as()` method.

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
from app.User import User
...

    def setUpFactories(self):
        User.create({
            'name': 'Joe',
            'email': 'user@example.com',
            'password': '$2b$12$WMgb5Re1NqUr.uSRfQmPQeeGWudk/8/aNbVMpD1dR.Et83vfL8WAu',  # == 'secret'
        })

    def test_user_can_see_dashboard(self):
        self.assertTrue(
            self.acting_as(User.find(1)).get('/dashboard').ok()
        )
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Passing in Data

Maybe you need to check a post request and pass in some input data like submitting a form. You can do this by passing in a dictionary as the second value to either the `get` or `post` method:

```python
def test_user_can_see_dashboard(self):
    self.assertTrue(
        self.acting_as(User.find(1)).post('/dashboard', {
            'name': 'Joe',
            'active': 1
        })
    )
```

The same can be applied to the get method except it will be in the form of query parameters.

### Test Example

To complete our test, let's check if the user is actually created:

```python
from masonite.testing import TestCase
from app.User import User

class TestUser(TestCase):

    """Start and rollback transactions for this test
    """
    transactions = True

    def setUp(self):
        super().setUp()

    def setUpFactories(self):
        User.create({
            'name': 'Joe',
            'email': 'user@example.com',
            # == 'secret'
            'password': '$2b$12$WMgb5Re1NqUr.uSRfQmPQeeGWudk/8/aNbVMpD1dR.Et83vfL8WAu',  
        })

    def test_creates_users(self):
        self.assertTrue(User.find(1))
```

Thats it! This test will now check that the user is created properly

### Running Tests

You can run tests by running:

```text
$ python -m pytest
```

