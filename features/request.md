The request and the response in Masonite work together to form a well formed response that a browser can read and render. The Request class is used to fetch any incoming cookies, headers, URL's, paths, request methods and other incoming data.

# Cookies

> **Although both the request and response classes have headers and cookies, in most instances, when fetching cookies and headers, it should be fetched from the Request class. When setting headers and cookies it should be done on the response class.**

To get cookies you can do so simply:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  request.cookie('Accepted-Cookies')
```

This will fetch the cookie from the incoming request headers.

You can also set cookies on the request:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  request.cookie('Accepted-Cookies', 'True')
```

> **Note that setting cookies on the request will NOT return the cookie as part of the response and therefore will NOT keep the cookie from request to request.**

# Headers

> Although both the request and response classes have headers and cookies, in most instances, when fetching cookies and headers, it should be fetched from the Request class. When setting headers and cookies it should be done on the response class.

To get a request header you can do so simply:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  request.header('X-Custom')
```

You can also set a header on the request class:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  request.header('X-Custom', 'value')
```

> **Note that setting headers on the request will NOT return the header as part of the response.**

# Paths

You can get the current request URI:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  request.get_path() #== /dashboard/1
```

You can get the current request method:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  request.get_request_method() #== PUT
```



You can check if the current path contains a glob style schema:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # URI: /dashboard/1/users
  request.contains("/dashboard/*/users") #== True
```

You can get the current subdomain from the host:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # URI: work.example.com
  request.get_subdomain() #== work
```

You can get the current host:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # URI: work.example.com
  request.get_host() #== example.com
```

# Inputs

Inputs can be from any kind of request method. In Masonite, fetching the input is the same no matter which request method it is.

To fetch an input we can do:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # GET /dashboard?user_id=1
  request.input('user_id') #== 1
```

If an input doesn't exist you can pass in a default:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # GET /dashboard
  request.input('user_id', 5) #== 5
```

To fetch all inputs from request we can do:

```python
from masonite.request import Request

def show(self, request: Request):
  request.all() #== {"user_id": 1, "name": "John", "email": "john@masoniteproject.com"}
```

Or we can only fetch some inputs:

To fetch all inputs from request we can do:

```python
from masonite.request import Request

def show(self, request: Request):
  request.only("user_id", "name") #== {"user_id": 1, "name": "John"}
```
### Getting Dictionary Input

If your input is a dictionary you have two choices how you want to access the dictionary.Take this code example:

```python
"""
Payload: 
{
"user": {
    "id": 1,
    "addresses": [
        {"id": 1, "street": "A Street"},
        {"id": 2, "street": "B Street"}
    ],
	"name":{
		"first":"user",
		"last":"A"
	}
 }
}
"""
```

You can either access it normally:

```python
request.input('user')['name']['last'] # A
```
Or you can use dot notation to fetch the value for simplicity:

```python
request.input('user.name.last') # A
```
You can also use a \* wildcard to get all values from a dictionary list.:
```python
request.input('user.addresses.*.id') # [1,2]
```

# Route Params

Route parameters are parameters specified in a route and captured from the URL:

If you have a route like this:

```python
Route.get('/dashboard/@user_id', 'DashboardController@show')
```

You can get the value of the `@user_id` parameter like this:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # GET /dashboard?user_id=1
  request.param('user_id') #== 1
```

# User

As a convenient way to fetch the user, you can do so directly on the request class if a user is logged in:

```python
from masonite.request import Request
#..

def show(self, request: Request):
  # GET /dashboard?user_id=1
  request.user() #== <app.User.User>
```

If the user is not authenticated then this will be set to `None`


# IP Address

You can fetch the IP address (ipv4) from which the request has been made by adding the `IpMiddleware`
to the HTTP middlewares of the project:

```python
# Kernel.py
from masonite.middleware import IpMiddleware

class Kernel:
    http_middleware = [
      # ...
      IpMiddleware
    ]
```

Then you can use the `ip()` helper:

```python
def show(self, request: Request):
    request.ip()
```

IP address is retrieved from various HTTP request headers in the following order:

- HTTP_CLIENT_IP
- HTTP_X_FORWARDED_FOR
- HTTP_X_FORWARDED
- HTTP_X_CLUSTER_CLIENT_IP
- HTTP_FORWARDED_FOR
- HTTP_FORWARDED
- REMOTE_ADDR

The first non-private and non-reserved IP address found by resolving the different headers will
be returned by the helper.

The headers used and their order can be customized by overriding the `IpMiddleware` and changing
the `headers` attribute:

```python
class CustomIpMiddleware(IpMiddleware):
    headers = [
      "HTTP_CLIENT_IP",
      "REMOTE_ADDR"
    ]
```
