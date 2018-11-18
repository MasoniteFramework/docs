# Responses

## Responses

## Introduction

Responses are primarily handled internally with Masonite but Masonite 2.1 brings a new `Response` object into the core framework. It is typically used to condense a lot of redundant logic down throughout the framework like getting the response ready, status codes, content lengths and content types.

Previously this needed to be individually set but now the response object abstracts a lot of the logic. You will likely never need to encounter this object during normal development but it is documented here if you need to use it similarly to how we use it in core.

### JSON Responses

We can set a JSON response by using the `json()` method. This simply requires a dictionary:

```python
from masonite.response import Response

def show(self, response: Response):
    return response.json({'key': 'value'})
```

This will set the `Content-Type`, `Content-Length`, status code and the actual response for you.

Keep in mind this is the same thing as doing:

```python
def show(self):
    return {'key': 'value'}
```

Since Masonite uses a middleware that abstracts this logic.

### View and Text Response

The `view()` method either takes a `View` object or a string:

```python
from masonite.response import Response
from masonite.view import View

def show(self, response: Response, view: View):
    return response.view('hello world')

def show(self, response: Response, view: View):
    return response.view(view.render('some.template'))
```

### Redirecting

You can also use some very basic URL redirection using the response object:

```python
from masonite.response import Response

def show(self, response: Response):
    return response.redirect('/some/route')
```

