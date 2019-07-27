# Testing

## Testing

## Introduction

Masonite testing is very simple. You can test very complex parts of your code with ease by just extending your class with a Masonite unit test class.

Although Masonite uses `pytest` to run tests, Masonite's test suite is based on `unittest`. So you will use `unittest` syntax but run the tests with Pytest. Because of this, all syntax will be in `camelCase` instead of PEP 8 `under_score`. Just know that all `TestCase` method calls used during testing is in `camelCase` form to maintain unittest standards.

Normal tests should still be underscore and start with `test_` though like this:

```python
def test_user_can_login(self):
    pass
```

## Configuration

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

## Environments

Most times you want to develop and test on different databases. Maybe you develop on a local MySQL database but your tests should run in a SQLlite database.

You can create a `.env.testing` file and put all database configs in that. When Pytest runs it will additionally load and override any additional environment variables.

Your `.env.testing` file may look like this:

```text
DB_CONNECTION=sqlite
DB_HOST=127.0.0.1
DB_DATABASE=masonite.db
DB_LOG=False

STRIPE_CLIENT=test_sk-9uxaxixjsxjsin
STRIPE_SECRET=test_sk-suhxs87cen88h7
```

Feel free to load any testing environment variables in here. By default they will not be commited.

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

### Method options

You can choose anyone of the normal request methods:

```python
def test_route_exists(self):
    self.get('/testing')
    self.post('/testing')
    self.put('/testing')
    self.patch('/testing')
    self.delete('/testing')
```

### JSON Requests

You can use a standard JSON request and specify whichever option you need using the `json()` method:

```python
def test_route_exists(self):
    self.json('POST', '/testing', {'user': 'Joe'})
```

### Testing If Route Has The Correct Name

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_route_has_the_correct_name(self):
    self.assertTrue(self.get('/testing').isNamed('testing.route'))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Testing If A Route Has The Correct Middleware

{% code-tabs %}
{% code-tabs-item title="tests/test\_unit.py" %}
```python
def test_route_has_route_middleware(self):
    assert self.get('/testing').hasMiddleware('auth', 'owner')
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

## CSRF Protection

By default, all calls to your routes with the above methods will be without CSRF protection. The testing code will allow you to bypass that protection.

This is very useful since you don't need to worry about setting CSRF tokens on every request but you may want to enable this protection. You can do so by calling the `withCsrf()` method on your test.

```python
def test_csrf(self):
    self.withCsrf()

    self.post('/unit/test/json', {'test': 'testing'})
```

This will enable it on a specific test but you may want to enable it on all your tests. You can do this by adding the method to your `setUp()` method:

```python
def setUp(self):
    super().setUp()
    self.withCsrf()

def test_csrf(self):
    self.post('/unit/test/json', {'test': 'testing'})
```

## Exception Handling

As you have noticed, Masonite has exception handling which it uses to display useful information during development.

This is an issue during testing because we wan't to be able to see more useful testing related issues. Because of this, testing will disable Masonite's default exception handling and you will see more useful exceptions during testing. If you want to use Masonite's built in exception handling then you can enable it by running:

```python
def setUp(self):
    super().setUp()
    self.withExceptionHandling()

def test_csrf(self):
    self.post('/unit/test/json', {'test': 'testing'})
```

## Getting Output

You can get the output by using the capture output easily by calling the `captureOutput` method on your unit test:

```python
def test_get_output(self):
    with self.captureOutput() as o:
        print('hello world!')

    self.assertEqual(o, 'hello world!')
```

## Testing JSON

A lot of times you will want to build tests around your API's. There are quite a few methods for testing your endpoints

### Testing the count

You can test to make sure your endpoint returns a specific amount of something. Like returning 5 articles:

```python
def test_has_articles(self):
    self.assertTrue(
        self.json('GET', '/api/articles').count(5)
    )
```

You can also use `amount` which is just an alias for `count`:

```python
def test_has_articles(self):
    self.assertTrue(
        self.json('GET', '/api/articles').amount(5)
    )
```

### Checking Amount of Specific Key

You can also check if a specific JSON key has a specific amount. For example:

```python
"""
{
    "name": "Joe",
    "tags": ['python', 'framework'],
    "age": 25
}
"""

def test_has_several_tags(self):
    self.assertTrue(
        self.json('GET', '/api/user').hasAmount('tags', 2)
    )
```

### Testing Specific Responses

Sometimes you will want to check if your endpoint returns some specific JSON values:

```python
"""
{
    "name": "Joe",
    "title": "creator",
    "age": 25
}
"""

def test_has_age(self):
    self.assertTrue(
        self.json('GET', '/api/user').hasJson('age', 25)
    )
```

You can also specify a dictionary of values as well and will check if each value inside the response:

```python
"""
{
    "name": "Joe",
    "title": "creator",
    "age": 25
}
"""

def test_has_age(self):
    self.assertTrue(
        self.json('GET', '/api/user').hasJson({
            "name": "Joe",
            "age": 25
        })
    )
```

You do not have to specify all of the endpoints, just the ones you want to check for.

### Dot Notation

You can also use dot notation for multi dimensional endpoints:

```python
"""
{
    "profile": {
        "name": "Joe",
        "title": "creator",
        "age": 25
    }
}
"""

def test_has_name(self):
    self.assertTrue(
        self.json('GET', '/api/user').hasJson('profile.name', 'Joe')
    )
```

### Testing Parameters

You can test if a specific parameter contains a specific value. For example if you want to see if the parameter `id` is equal to `5`:

```python
def test_has_name(self):
    # Route is: /dashboard/user/@id
    self.assertTrue(
        self.get('GET', '/dashboard/user/5').parameterIs('id', '5')
    )

    self.get('GET', '/dashboard/user/5').assertParameterIs('id', '5')
```

### Testing Headers

You can test if a specific header contains a specific value. For example if you want to see if the header `Content-Type` is equal to `text/html`:

```python
def test_has_name(self):
    # Route is: /dashboard/user/@id
    self.assertTrue(
        self.get('GET', '/dashboard/user/5').headerIs('Content-Type', 'text/html')
    )
   
    self.get('GET', '/dashboard/user/5').assertHeaderIs('Content-Type', 'text/html')
```

### Subdomains

By default, Masonite turns off subdomains since this can cause issues when deploying to a PaaS that deploys to a subdomain like `sunny-land-176892.herokuapp.com` for example. 

To activate subdomains in your tests you will have to use the `withSubdomains()` method. You can then set the host in the `wsgi` attribute.

```python
def test_subdomains(self):
    self.withSubdomains().get('/view', wsgi={
            'HTTP_HOST': 'subb.domain.com'
        }).assertIsStatus(404)
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

## Users

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
            self.actingAs(User.find(1)).get('/dashboard').ok()
        )
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Passing in Data

Maybe you need to check a post request and pass in some input data like submitting a form. You can do this by passing in a dictionary as the second value to either the `get` or `post` method:

```python
def test_user_can_see_dashboard(self):
    self.assertTrue(
        self.actingAs(User.find(1)).post('/dashboard', {
            'name': 'Joe',
            'active': 1
        })
    )
```

The same can be applied to the get method except it will be in the form of query parameters.

## Test Example

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

## Running Tests

You can run tests by running:

```text
$ python -m pytest
```

