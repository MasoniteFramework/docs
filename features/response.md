The request and the response in Masonite work together to form a well formed response that a browser can read and render. The Response class is responsible for what data is returned and how it is formatted. In this case, a response can have cookies and headers. The Request class can also have cookies and headers. In most times, generally, when you have to fetch headers or cookies you should do so on the request class and when you set cookies and headers you should do so on the response class.

# Cookies

Cookies on the response class are attached to the response and rendered as part of the response headers. Any incoming cookies you would likely fetch from the request class but you can fetch them from the response if you have a reason to:

```python
from masonite.response import Response

def show(self, response: Response):
    response.cookie("key")
```

You can also set cookies on the response:

```python
from masonite.response import Response

def show(self, response: Response):
    response.cookie("Accepted-Cookies", "True")
```

You can also delete cookies:

```python
from masonite.response import Response

def show(self, response: Response):
    response.delete_cookie("Accepted-Cookies")
```

# Redirecting

You can redirect the user to any number of URL's.

First you can redirect to a simple URL:

```python
from masonite.response import Response

def show(self, response: Response):
    response.redirect('/home')
```

You can redirect back to where the user came from:

```python
from masonite.response import Response

def show(self, response: Response):
    response.back()
```

If using the back method as part of a form request then you will need to use the back view helper as well:

```html
<form>
	{{ csrf_field }}
  {{ back() }}
  <input type="text">
  ...
</form>
```

You can also redirect to a route by its name:

```python
from masonite.response import Response

def show(self, response: Response):
    response.redirect(name='users.home')
```

This will find a named route `users.home` like:

```python
Route.get('/dashboard/@user_id', 'DashboardController@show').name('users.home')
```

You may also pass parameters to a route if the URL requires it:

```python
from masonite.response import Response

def show(self, response: Response):
    response.redirect(name='users.home', {"user_id", 1})
```

Finally you may also pass query string parameters to the url or route to redirect:

```python
def show(self, response: Response):
    response.redirect(name='users.home', {"user_id", 1}, query_params={"mode": "preview"})
#== will redirect to /dashboard/1?mode=preview
```

# Headers

Response headers are any headers that will be found on the response. Masonite takes care of all the important headers for you but there are times where you want to set you own custom headers.

Most incoming headers you want to get can be found on the request class but if you need to get any headers on the response:

```python
from masonite.response import Response

def show(self, response: Response):
	response.header('key')
```

Any responses you want to be returned should be set on the response class:

```python
from masonite.response import Response

def show(self, response: Response):
	response.header('X-Custom', 'value')
```

This will set the header on the response.

# Status

You can set the status on the response simply:

```python
from masonite.response import Response

def show(self, response: Response):
	response.status(409)
```

# Downloading

You can also very simply download assets like images, PDF's or other files:

```python
def show(self, response: Response):
  return response.download("invoice-2021-01", "path/to/invoice.pdf")
```

This will set all the proper headers required and render the file in the browser.

> When setting the name, the file extension will be picked up from the file type. This example will download the invoice as the name `invoice-2021-01.pdf`

If you want to force download it, or automatically download the file when the response in rendered, you can add a `force` parameter and set it to `True`:

```python
def show(self, response: Response):
    return response.download("invoice-2021-01", "path/to/invoice.pdf", force=True)
```

