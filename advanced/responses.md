# Responses

# Controller Methods

Most of the responses you will work with simply involve returning various data types / classes / objects in the controller method. For example, you may be used to returning a `view.render()` object in the controller method. This will return a `View` instance which Masonite will extract out the rendered html template from it.

Below is a list of all the responses you can return

## Strings

You can simply return a string which will output the string to the browser:

```python
def show(self):
    return 'string here'
```

This will set headers and content lengths similiar to a normal HTML response.

## Views

You can return an instance of a `View` object which Masonite will then pull the HTML information that Jinja has rendered. This is the normal process of returning your templates. You can do so by type hinting the view class and using the render method:

```python
from masonite.view import View

def show(self, view: View):
    return view.render('your/template', {'key': 'value'})
```

Notice you can also pass in a dictionary as a second argument which will pass those variables to your Jinja templates.

## JSON (Dictionaries)

There are a few ways to return JSON responses. The easiest way is to simply return a dictionary like this:

```python
def show(self):
    return {'key': 'value'}
```

This will return a response with the appropriate JSON related headers.

## JSON (Models)

If you are working with models then its pretty easy to return a model as a JSON response by simply returning a model. This is useful when working with single records:

```python
from app.User import User
# ...

def show(self):
    return User.find(1)
```

This will return a response like this:


```json
{
    "id": 1,
    "name": "Brett Huber",
    "email": "hubbardtimothy@gmail.com",
    "password": "...",
    "remember_token": "...",
    "verified_at": null,
    "created_at": "2019-08-24T01:26:42.675467+00:00",
    "updated_at": "2019-08-24T01:26:42.675467+00:00",
}
```

## JSON (Collections)

If you are working with collections you can return something similiar which will return a slightly different JSON response with several results:

```python
from app.User import User
# ...

def show(self):
    return User.all()
```

Which will return a response like:

```json
[
    {
        "id": 1,
        "name": "Brett Huber",
        "email": "hubbardtimothy@gmail.com",
        "password": "...",
        ...
    },
    {
        "id": 2,
        "name": "Jack Baird",
        "email": "phelpsrebecca@stanley.info",
        "password": "...",
        ...
    },
    ...
    }
]
```

## JSON (Pagination)

If you need to paginate a response you can return an instance of `Paginator`. You can do so easily by using the `paginate()` method:

```python
from app.User import User
# ...

def show(self):
    return User.paginate(10)
```

The value you pass in to the paginate method is the page size or limit of results you want to return.

This will return a response like:

```json
{
    "total": 55,
    "count": 10,
    "per_page": 10,
    "current_page": 1,
    "last_page": 6,
    "from": 1,
    "to": 10,
    "data": [
        {
            "id": 1,
            "name": "Brett Huber",
            "email": "hubbardtimothy@gmail.com",
            "password": "...",
            ...
        },
        {
            ...
        }
```

You can override the page size and page number by passing in the appropriate query inputs. You can change the page you are looking at by passing in a `?page=` input and you can change the amount of results per page by using the `?page_size=` input. 

If you are building an API this might look like `/api/users?page=2&page_size=5`. This will return 5 results on page 2 for this endpoint.



# Response Class

The response class is what Masonite uses internally but you can explicit use it if you find the need to. A need might include setting a response in a middleware or a service provider where Masonite does not handle all the response converting for you. It is typically used to condense a lot of redundant logic down throughout the framework like getting the response ready, status codes, content lengths and content types.

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

