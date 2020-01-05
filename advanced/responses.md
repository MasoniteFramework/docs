# Responses


# Introduction

Responses are primarily handled internally with Masonite but Masonite 2.1 brings a new `Response` object into the core framework. It is typically used to condense a lot of redundant logic down throughout the framework like getting the response ready, status codes, content lengths and content types.

Previously this needed to be individually set but now the response object abstracts a lot of the logic. You will likely never need to encounter this object during normal development but it is documented here if you need to use it similarly to how we use it in core.

# JSON Responses

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

# View and Text Response

The `view()` method either takes a `View` object or a string:

```python
from masonite.response import Response
from masonite.view import View

def show(self, response: Response, view: View):
    return response.view('hello world')

def show(self, response: Response, view: View):
    return response.view(view.render('some.template'))
```

# Redirecting

You can also use some very basic URL redirection using the response object:

```python
from masonite.response import Response

def show(self, response: Response):
    return response.redirect('/some/route')
```

# Responsable Classes

Responsable classes are classes that are allowed to be returned in your controller methods. These classes simply need to inherit a `Responsable` class and then contain a `get_response` method.

Let's take a look at a simple hello world example:

```python
from masonite.response import Responsable

class HelloWorld(Responsable):

    def get_response(self):
        return 'hello world'
```

This class can now be returned in a controller method

```python
from some.place import HelloWorld

def show(self):
    return HelloWorld()
```

Masonite will check if the response is an instance of `Responsable` and run the `get_response` method. This will show "Hello world" to the browser. This is actually how Masonites view class and mail classes work so you can see how powerful this can be.

# Mailables

You can also return mailables. This is great if you want to debug what your emails will look like before you send them. You can do so by simply returning the mailable method of the mail class:

```python
from app.mailables import WelcomeEmail

def show(self, mail: Mail):
    return mail.mailable(WelcomeEmail())
```

This will now show what the email will look like.