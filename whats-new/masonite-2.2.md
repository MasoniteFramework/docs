# Masonite 2.2

## Masonite 2.2

{% hint style="danger" %}
This release is still in beta and is not yet released. All information in this documentation section is subject to change.
{% endhint %}

## Route Prefixes

Previously you had to append all routes with a `/` character. This would look something like:

```python
Get('/some/url')
```

You can know optionally prefix this without a `/` character:

```python
Get('some/url')
```

## URL parameters can now optionally be retrieved from the controller definition

Previously we had to do something like:

```python
def show(self, view: View, request: Request):
    user = User.find(request.param('user_id'))
    return view.render('some.template', {'user': user})
```

Now we can **optionally** get the parameter from the method definition:

```python
def show(self, user_id, view: View):
    user = User.find(user_id)
    return view.render('some.template', {'user': user})
```

## Added a storage manager and disk storage drivers

This is used as a wrapper around I/O operations. It will also be a wrapper around the upload drivers and moving files around and other file management type operations

## Async driver now can be specified whether to use threading or processing

We can now specify directly in the configuration file whether or not the threading or multiprocessing for the async type operations.

## Added new HTTP Verbs

We added 4 new HTTP verbs: HEAD, CONNECT, OPTIONS, TRACE. You import these and use them like normal:

```python
from masonite.routes import Connect, Trace
ROUTES = [
    Connect('..'),
    Trace('..'),
]
```

## JSON error responses

If the incoming request is a JSON request, Masonite will now return all errors as JSON

```javascript
{
  "error": {
    "exception": "Invalid response type of <class 'set'>",
    "status": 500,
    "stacktrace": [
        "/Users/joseph/Programming/core/bootstrap/start.py line 38 in app",
        "/Users/joseph/Programming/core/masonite/app.py line 149 in resolve",
        "/Users/joseph/Programming/core/masonite/providers/RouteProvider.py line 92 in boot",
        "/Users/joseph/Programming/core/masonite/response.py line 105 in view"
    ]
  }
}
```

## Rearranged Drivers into their own folders

This is more of an internal change for Core itself.

