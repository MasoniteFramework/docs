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

## Database tests

## Console tests

## HTTP tests

## Assertions

## Mocks

## Extending test response
