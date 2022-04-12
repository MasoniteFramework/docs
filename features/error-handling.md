Masonite error handling is based on the `ExceptionHandler` class which is responsible for handling all exceptions thrown by your application.

# Global Exception Handler

All exceptions are handled by the ExceptionHandler class which is bound to the [Service Container](../architecture/service-container.md) through `exception_handler` key.

This handler has a logic to decide how to handle exceptions depending on the exception type, the environment type, the request accepted content type and the [configured exception handlers](#adding-new-handlers).

This handler has by default one driver [Exceptionite](https://github.com/MasoniteFramework/exceptionite) which is responsible of handling errors in development by providing a lot of context to help debug your application.

## Debug Mode

When [Debug mode](/features/environments.md#debug-mode) is enabled all exceptions are rendered through Exceptionite HTML debug page.

When disabled, the default `errors/500.html`, `errors/404.html`, `errors/403.html` error pages are rendered depending on error type.

{% hint style="warning" %}
Never deploy an application in production with debug mode enabled ! This could lead to expose some
sensitive configuration data and environment variables to the end user.
{% endhint %}

## Lifecycle

When an exception is raised it will be caught by the ExceptionHandler. Then the following cycle will happen:

### In Development

1. A `masonite.exception.SomeException` event will be fired.
2. A specific ExceptionHandler will be used if it exists for the given exception.
3. Exceptionite will then handle the error by rendering it in the console
    a. and with the Exceptionite JSON response if accepted content is `application/json`
    b. else with the Exceptionite HTML error page

### In Production

1. A `masonite.exception.SomeException` event will be fired.
2. A specific ExceptionHandler will be used if it exists for the given exception. In this case the default ExceptionHandler won't process the exception anymore.
3. If exception is an [HTTP exception](#http-exceptions) it will be handled by the HTTPExceptionHandler which
is responsible for selecting an error template (`errors/500.html`, `errors/404.html`, `errors/403.html`) and rendering it.
4. If exception is a [Renderable exception](#renderable-exceptions) it will be rendered accordingly.
5. Else this exception has not been yet handled and will be handled as an HTTP exception with 500 Server Error status code.


# Report Exceptions

To report an exception you should simply raise it as any Python exceptions:

```python
def index(self, view:View):
    user = User.find(request.param("id"))
    if not user:
        raise RouteNotFoundException("User not found")
```

There are different type of exceptions.

## Simple Exceptions

Simple exceptions will be handled as a 500 Server Error if no [custom exceptions handlers](#adding-new-handlers) are
defined for it.

## HTTP Exceptions (TODO)

Explain options and how they will be rendered

If template is not found 500 error template will be used instead to ensure your error is still rendered.
List HTTP exceptions here
- PermissionDenied (403)
- RouteNotFoundException, RecordNotFound (404)
- BadRequest (400)

Explain how to use options for those exceptions

Explain how to create a custom one

## Renderable Exceptions

If you want more flexibility to render your exception without using the HTTPExceptionHandler above, you can just add a `get_response()` method to it.
This method will be given as first argument of `response.view()`, so that you can render simple string or your own view template.

```python
class CustomException(Exception):

    def __init__(self, message=""):
        super().__init__(message)
        self.message = message

    def get_response(self):
        return self.message
```

Here when raising this exception in your code, Masonite will know that it should render it by calling the `get_response()` method and here it will render as a string containing the message raised.

Note that you can also set an HTTP status code by adding - as for HTTP exceptions - the `get_status()` method to the exception.

```python
class CustomException(Exception):

    def __init__(self, message=""):
        super().__init__(message)
        self.message = message

    def get_response(self):
        return self.message

    def get_status(self):
        return 401
```

## Existing Exception Handlers

Existing Exception Handlers are:
- HTTPExceptionHandler
- DumpExceptionHandler

You can build your own Exception Handlers to override the way Masonite handles an exception that is thrown or to add new behaviours.

## Adding New Handlers

A handler is a simple class with a `handle()` method that Masonite will call when a specific exception is thrown by the application.

As an example, you could handle specifically `ZeroDivisionError` exceptions that could be thrown by your application. It will look like this:

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
from app.exceptions.DivideException import DivideException

self.application.bind(
    "ZeroDivisionErrorHandler",
    DivideException(self.application)
)
```

{% hint style="info" %}
You can add this binding in your AppProvider or in `Kernel.py`.
{% endhint %}

Now when your application throws a `ZeroDivisionError`, Masonite will use your handler rather than Masonite's own exception handlers.
