Masonite testing is very simple. You can test very complex parts of your code with ease by just extending your class with a Masonite unit test class.

Although Masonite uses `pytest` to run tests, Masonite's test suite is based on `unittest`. So you will use `unittest` syntax but run the tests with `pytest`. Because of this, all syntax will be in camelCase instead of PEP 8 lower_case_with_underscores. Just know that all TestCase assertions used during testing is in camelCase form to maintain unittest standards.

## Environment

When running tests, Masonite will automatically set the [environment](../features/environments.md) to `testing`. You are free to define other testing environment configuration values as necessary.

You can create a `.env.testing` file. Feel free to load any testing environment variables in here. By default they will not be commited. When `pytest` runs it will additionally load and override any additional environment variables.


## Creating Tests

You can simply create a file starting with `test_` and then creating a test class inheriting from masonite `TestCase` class.

You can also directly use the command

```python
$ python craft test SomeFeatureTest
```

to create `tests/unit/test_some_feature.py`:

```python
from masonite.tests import TestCase


class SomeFeatureTest(TestCase):
    def setUp(self):
        super().setUp()

    def test_something(self):
        self.assertTrue(True)
```

That's it! You're ready to start testing. Read on to learn how to build your test cases and run them.

## Running Tests

You can run tests by calling

```
$ python -m pytest
```

This will automatically discover your tests following pytest [automatic tests discovery](https://docs.pytest.org/en/6.2.x/goodpractices.html#test-discovery).
You can also run a specific test class

```
$ python -m pytest tests/unit/test_my_feature.py
```

Or a specific test method

```
$ python -m pytest tests/unit/test_my_feature.py::MyFeatureTest::test_feature_is_working
```

Finally you can re-run the last failed tests automatically

```
$ python -m pytest --last-failed
```

## Building Tests

### Test Life Cycle

When you run a test class each test method of this test class will be ran following a specific
life cycle.

```python
class TestFeatures(TestCase):

    @classmethod
    def setUpClass(cls):
        """Called once before all tests of this class are executed."""
        print("Setting up test class")

    @classmethod
    def tearDownClass(cls):
        """Called once after all tests of this class are executed."""
        print("Cleaning up test class")

    def setUp(self):
        """Called once before each test are executed."""
        super().setUp()
        print("Setting up individual unit test")

    def tearDown(self):
        """Called once after each test are executed."""
        super().tearDown()
        print("Cleaning up individual unit test")

    def test_1(self):
        print("Running test 1")

    def test_2(self):
        print("Running test 2")
```

Running the above test class will create this output:

```
Setting up test class
Setting up individual unit test
Running test 2
Cleaning up individual unit test
Setting up individual unit test
Running test 1
Cleaning up individual unit test
Cleaning up test class
```

{% hint style="warning" %}
Note that tests methods are not always ran in the order specified in the class. Anyway you should
not make the assumptions that tests will be run in a given order. You should try to make your
tests idempotent.
{% endhint %}

### Chaining assertions

All methods that begin with `assert` can be chained together to run through many assertions. All other method will return some kind of boolean or value which you can use to do your own assertions.

### Asserting exceptions

Sometimes you need to assert that a given piece of code will raise a given exception. To do this you
can use the standard `assertRaises()` context manager:

```python
with self.assertRaises(ValidationError) as e:
    # run some code here
    raise ValidationError("An error occured !")

self.assertEqual(str(e.exception), "An error occured !")
```

### Capturing test output

Sometimes you need to test the output of a function that prints to the console. To do this in your
tests you can use the `captureOutput()` context manager:

```python
with self.captureOutput() as output:
    # run some code here
    print("Hello World !")

self.assertEqual(output.getvalue().strip(), "Hello World !")
```

## Helpers

Masonite comes with different helpers that can ease writing tests. Some of them have already been explained in sections above.

- [withExceptionsHandling](#withExceptionsHandling)
- [withoutExceptionsHandling](#withoutExceptionsHandling)
- [withCsrf](#withCsrf)
- [withoutCsrf](#withoutCsrf)
- [withCookies](#withCookies)
- [withHeaders](#withHeaders)
- [fakeTime](#fakeTime)
- [fakeTimeTomorrow](#fakeTimeTomorrow)
- [fakeTimeYesterday](#fakeTimeYesterday)
- [fakeTimeInFuture](#fakeTimeInFuture)
- [fakeTimeInPast](#fakeTimeInPast)
- [restoreTime](#restoreTime)

### withExceptionsHandling

Enable exceptions handling during testing.

```python
self.withExceptionsHandling()
```

### withoutExceptionsHandling

Disable exceptions handling during testing.

```python
self.withoutExceptionsHandling()
```

{% hint style="warning" %}
Note that exception handling is disabled by default during testing.
{% endhint %}

### withCsrf

Enable CSRF protection during testing.

```python
self.withCsrf()
```

### withoutCsrf

Disable CSRF protection during testing.

```python
self.withoutCsrf()
```

{% hint style="warning" %}
Note that CSRF protection is disabled by default during testing.
{% endhint %}

### withCookies

Add cookies that will be used in the next request. This method accepts a dictionary of name / value pairs. Cookies dict is reset between each test.

```python
self.withCookies(data)
```

### withHeaders

Add headers that will be used in the next request. This method accepts a dictionary of name / value pairs. Headers dict is reset between each test.

```python
self.withHeaders(data)
```

### fakeTime

Set a given pendulum instance to be returned when a `now` (or `today`, `tomorrow` `yesterday`) instance is created. It's really useful during tests to check
timestamps logic.

This allow to control which datetime will be returned to be able to always have an expected behaviour in the tests.

```python
given_date = pendulum.datetime(2021, 2, 5)
self.fakeTime(given_date)
self.assertEqual(pendulum.now(), given_date)
```

{% hint style="warning" %}
When using those helpers you should not forget to reset the default `pendulum` behaviour with `restoreTime()` helper to avoid breaking other tests. It can be done directly in the test or in a `tearDown()` method.
{% endhint %}

### fakeTimeTomorrow

Set the mocked time as tomorrow. (It's a shortcut to avoid doing `self.fakeTime(pendulum.tomorrow())`).

```python
tomorrow = pendulum.tomorrow()
self.fakeTimeTomorrow()
self.assertEqual(pendulum.now(), tomorrow)
```

### fakeTimeYesterday

Set the mocked time as yesterday.

### fakeTimeInFuture

Set the mocked time as an offset of a given unit of time in the future. Unit can be specified among pendulum units: `seconds`, `minutes`, `hours`, `days` (default), `weeks`, `months`, `years`.

```python
self.fakeTimeInFuture(offset, unit="days")
```

```python
real_now = pendulum.now()
self.fakeTimeInFuture(1, "months")
self.assertEqual(pendulum.now().diff(real_now).in_months(), 1)
```

### fakeTimeInPast

Set the mocked time as an offset of a given unit of time in the past. Unit can be specified among pendulum units: `seconds`, `minutes`, `hours`, `days` (default), `weeks`, `months`, `years`.

```python
self.fakeTimeInPast(offset, unit="days")
```

### restoreTime

Restore the mocked time behaviour to default behaviour of `pendulum`. When using `fake` time helpers you should not forget to call this helper at the end.

It can be done directly in the test or in a `tearDown()` method.

```python
def tearDown(self):
    super().tearDown()
    self.restoreTime()

def test_creation_date(self):
    self.fakeTimeYesterday()
    # from now on until the end of this unit test, time is mocked and will return yesterday time
```
