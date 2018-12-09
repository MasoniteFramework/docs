# Testing

## Introduction

Masonite testing is very simple. You can test very complex parts of your code with ease by just extending your class with a Masonite unit test class.

You can run tests by running:

```bash
$ python -m pytest 
```

## Configuration

First, create a new test class in a testing directory. We will use the directory `tests/unit` for the purposes of this documentation. In that file we will create a `test_unit.py` file and put a simple class in it like so:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
class TestSomeUnit:
    pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We will then inherit the Masonite UnitTest class so we have access to several built in helpers:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
from masonite.testing import UnitTest

class TestSomeUnit(UnitTest):
    pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

That's it! You're ready to start testing.

{% hint style="warning" %}
Your unit test needs to start with `Test` in order for pytest to pick it up.
{% endhint %}

## Usage

### Setup method

In unit testing, there is something called a setup\_method. What this method does is it runs when the test class is first ran. This is good for setting up routes or anything else all your tests need.

If we modify the setup method, we need to call the parent classes setup method. This looks like this:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
from masonite.testing import UnitTest
from masonite.routes import Get

class TestSomeUnit(UnitTest):
    
    def setup_method(self):
        super().setup_method()
        
        self.routes([
            Get().route('/testing', 'TestController@show').name('testing.route').middleware('auth', 'owner')
        ])
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Notice here we are adding a mock route here that we can do some testing on. you might instead want to test your routes specifically:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
from masonite.testing import UnitTest
from routes.web import ROUTES

class TestSomeUnit(UnitTest):
    
    def setup_method(self):
        super().setup_method()
        
        self.routes(ROUTES)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now we are working with the routes in our specific project.

### Routes

We have a few options for testing our routes. We will use a single route here so we can e

#### Testing If a Route Exists:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
from masonite.testing import UnitTest
from routes.web import ROUTES

class TestSomeUnit(UnitTest):
    
    def setup_method(self):
        super().setup_method()
        
        self.routes([
            Get().route('/testing', 'SomeController@show').name('testing.route').middleware('auth', 'owner')
        ])
    
    def test_route_exists(self):
        assert self.route('/testing')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Testing If Route Has The Correct Name

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_route_has_the_correct_name(self):
    assert self.route('/testing').is_named('testing.route')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Testing If A Route Has The Correct Middleware

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_route_has_route_middleware(self):
    assert self.route('/testing').has_middleware('auth', 'owner')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Testing If A Route Has The Correct Controller

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
from app.http.controllers.SomeController

def test_unit_test_has_controller(self):
    assert self.route('/testing').has_controller(SomeController)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Testing If A Route Contains A String

This can be used to see if the template returned a specific value

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_view_contains(self):
    assert self.route('/testing').contains('Login')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Status Code

You can also check if a route is "ok" or a status 200:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_view_is_status_200(self):
    assert self.route('/testing').status('200 OK')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or a shorthand:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_view_is_ok(self):
    assert self.route('/testing').ok()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### JSON

You can also test the result of a JSON response:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_json_response(self):
    json = self.json('/test/json/response/1', {'id': 1}, method="POST")
    assert json.status('200 OK')
    assert json.contains('success')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Users

We can load users into the route and check if they can view the route. This is good to see if your middleware is acting good against various users. 

For example we can check if a user that isn't logged in has access to the dashboard homepage:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
def test_guest_user_can_view(self):
    assert not self.route('/some/protect/route').user(None).can_view()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Or we can set a value on a mock user and see if that passes:

{% code-tabs %}
{% code-tabs-item title="tests/unit/test\_unit.py" %}
```python
class MockUser:
    is_admin = 1

def test_owner_user_can_view(self):
    assert self.route('/some/protect/route').user(MockUser).can_view()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

