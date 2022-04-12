

# Error Handler


## Local / Production


# Report Exceptions


## Simple Exceptions

## HTTP Exceptions

## Renderable Exceptions

Exceptions can be handled in Masonite by creating "Exception Handlers". These are custom classes you can build to override the way Masonite handles an exception that is thrown.

## Adding New Handlers

A handler is a simple class with a `handle` method that Masonite will call when a specific exception is thrown by the application.

As an example Exception Handler when your application throws a `ZeroDivisionError` for example will look like this:


```python
class DivideException:

    def __init__(self, application)
        self.application = application

    def handle(self, exception):
        self.application.make('response').view({
            "error": str(exception),
            "message": "You cannot divide by zero"
        }, status=500)
```

The name of the class can be whatever you like. In the handle method you should manually return a response by using the response class.

You will then need to register the class to the container using a specific key binding. The key binding will be `{exception_name}Handler`. You can do this in your Kernel file.

To register a custom exception handler for our `ZeroDivisionError` we would create a binding that looks like this:

```python
# Kernel.py
from app.exceptions.DivideException import DivideException

self.application.bind(
    "ZeroDivisionErrorHandler",
    DivideException(self.application)
)
```

Now when your application throws a `ZeroDivisionError`, Masonite will use your handler rather than Masonite's own exception handlers.
