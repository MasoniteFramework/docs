Masonite testing is very simple. You can test very complex parts of your code with ease by just extending your class with a Masonite unit test class.

Although Masonite uses `pytest` to run tests, Masonite's test suite is based on `unittest`. So you will use `unittest` syntax but run the tests with `pytest`. Because of this, all syntax will be in camelCase instead of PEP 8 lower_case_with_underscores. Just know that all TestCase assertions used during testing is in camelCase form to maintain unittest standards.

## Getting Started

### Environment

When running tests, Masonite will automatically set the environment to `testing`. You are free to define other testing environment configuration values as necessary.

You can create a `.env.testing` file. Feel free to load any testing environment variables in here. By default they will not be commited. When `pytest` runs it will additionally load and override any additional environment variables.

### Creating tests

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

### Running tests

You can run tests by calling

```
$ python -m pytest
```

This will automatically discover your tests following pytest [automatic tests discovery]().
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

## Building test cases

- Explain setUp / tearDown classes

## HTTP tests

### CSRF protection

### Exception handling

## Console tests

You can test [commands](/features/commands) running in console with `craft` test helper.

```python
def test_my_command(self):
    self.craft("my_command", "arg1", "arg2").assertSuccess()
```

This will programmatically run the command if it has been registered in your project and assert that no
errors has been reported.

More [console assertions](#console-assertions) are available.

## Database tests

## Available Assertions

### Responses Assertions

- assertContains
- assertNotContains
- assertContainsInOrder
- assertIsNamed
- assertIsNotNamed
- assertIsStatus
- assertNotFound
- assertOk
- assertCreated
- assertSuccessful
- assertNoContent
- assertUnauthorized
- assertHasHeader
- assertHeaderMissing
- assertLocation
- assertRedirect
- assertCookie
- assertPlainCookie
- assertCookieExpired
- assertCookieNotExpired
- assertSessionHas
- assertSessionMissing
- assertSessionHasErrors
- assertSessionHasNoErrors
- assertViewIs
- assertViewHas
- assertViewHasExact
- assertViewMissing
- assertAuthenticated
- assertGuest
- assertAuthenticatedAs
- assertHasHttpMiddleware
- assertHasRouteMiddleware
- assertHasController
- assertRouteHasParameter
- assertJson
- assertJsonPath
- assertJsonExact
- assertJsonCount
- assertJsonMissing

### Console Assertions

- assertSuccess
- assertExactOutput
- assertHasErrors
- assertExactErrors
- assertOutputContains
- assertOutputMissing

### Database Assertions

- assertDatabaseCount
- assertDatabaseHas
- assertDatabaseMissing
- assertDeleted
- assertSoftDeleted

## Helpers

- withExceptionsHandling
- withExceptionsHandling
- withCsrf
- withoutCsrf
- withCookies
- withHeaders
- fakeTime
- fakeTimeTomorrow
- fakeTimeYesterday
- fakeTimeInFuture
- fakeTimeInPast
- restoreTime

## Mocks

### Masonite features mocks

Using helpers for different features to list

- fake()
- restore()

### Basic Python mocks

- quick example external api
- link to docs of mock module

## Extending test response
