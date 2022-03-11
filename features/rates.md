# Rate Limiting

Masonite includes a rate limiting feature which make it really easy to limit an action during a given time window.

The most frequent use of this feature is to add throttling to some API endpoints but you could also
use this to limit how many time a model can be updated, a job can be run in the queue or a mail should be send.

This feature is based on the application [Cache](/cache.md). It will store the number of attempts of a given `key` and will also
store the associated time window.


## Overview

Rate Limiting feature can be accessed via the container `application.make("rate")` or by using the `RateLimiter` facade.

To limit an action, we need:

- a `key` that will uniquely identify the action
- a number of authorized attempts
- a delay after which number of attempts will be reset


## Defining Limits

To use Rate Limiting feature we should define limits. A limit is a number of attempts during a given time frame. It could
be 100 times per day or 5 times per hour. In Masonite limits are abstracted through the `Limit` class.

Here is an overview of the different ways to create limits:
```python
from masonite.rates import Limit

Limit.from_str("100/day")
Limit.per_day(100)  # equivalent to above

Limit.per_minute(10)
Limit.per_hour(5)
Limit.unlimited()  # to define an unlimited limit
```

Then to associate a given key to this limit we can do:
```python
username = "sam"
Limit.per_hour(5).by(username)
```

## Limit an action

This feature allows to simply limit calling a Python callable. Here we will limit the given callable to be called 3 times per hour for the key `sam`.

Let's make one attempt:
```python
def send_welcome_mail():
    # ...

RateLimiter.attempt("sam", send_welcome_mail, max_attempts=3, delay=60*60)
```

We can get the number of attempts:
```python
RateLimiter.attempts("sam") #== 1
```

We can get the number of remaining attempts:
```python
RateLimiter.remaining("sam", 3) #== 2
```

We can check if too many attempts have been made:

```python
if RateLimiter.too_many_attempts("sam", 3):
    print("limited")
else:
    print("ok")
```

We can reset the number of attempts:
```python
RateLimiter.reset_attempts("sam")
RateLimiter.attempts("sam") #== 0
```

We can get the seconds in which will be able to attempt the action again:
```python
RateLimiter.available_in("sam") #== 356
```

We can get the UNIX timestamps in seconds when will be able to attempt the action again:
```python
RateLimiter.available_at("sam") #== 1646998321
```


## Throttle HTTP Requests

You can add throttling to HTTP requests easily by using the `ThrottleRequestsMiddleware`.

First register the middleware as a route middleware into your project:

```python
# Kernel.py
from masonite.middleware import ThrottleRequestsMiddleware

class Kernel:
    route_middleware = {
        "web": [
            SessionMiddleware,
            LoadUserMiddleware,
            VerifyCsrfToken,
        ],
        "throttle": [ThrottleRequestsMiddleware],
    }
```

This middleware is taking one argument which can be either a limit string or a limiter name.

### Using Limit Strings

By using limit strings such as `100/day` you will be able to add a **global** limit which does not link this limit to a given user, view or IP address. It's really an absolute limit that you will define on your HTTP requests.

Units available are: `minute`, `hour` and `day`.

We now just have to use it in our routes:
```python
# web.py
Route.post("/api/uploads/", "UploadController@create").middleware("throttle:100/day")
Route.post("/api/videos/", "UploadController@create").middleware("throttle:10/minute")
```

### Using Limiters

To be able to add throttling per users or to implement more complex logic you should use `Limiter`. `Limiter` are simple classes with a `allow(request)` method that will be called each time a throttled HTTP request is made.

Some handy limiters are bundled into Masonite:

- GlobalLimiter
- UnlimitedLimiter
- GuestsOnlyLimiter

But you can create your own limiter:

```python
from masonite.rates import Limiter

class PremiumUsersLimiter(Limiter):

    def allow(self, request):
        user = request.user()
        if user:
            if user.role == "premium":
                return Limit.unlimited()
            else:
                return Limit.per_day(10).by(request.ip())
        else:
            return Limit.per_day(2).by(request.ip())
```

Here we are creating a limiter authorizing 2 requests/day for guests users, 10 requests/day for authenticated non-premium users and unlimited requests for premium users.
Here we are using `by(key)` to define how we should identify users. (TODO: explain more)

Finally you can register your limiter(s) in your application provider:

```python
from masonite.facades import RateLimiter
from masonite.rates import GuestsOnlyLimiter
from app.rates import PremiumUsersLimiter

class AppProvider(Provider):

    def register(self):
        RateLimiter.register("premium", PremiumUsersLimiter())
        # register an other limiter which gives unlimited access for authenticated users
        # and 2 request/day for guests users
        RateLimiter.register("guests", GuestsOnlyLimiter("2/hour"))
```

We now just have to use it in our routes:

```python
# web.py
Route.post("/api/uploads/", "UploadController@create").middleware("throttle:premium")
Route.post("/api/videos/", "UploadController@create").middleware("throttle:guests")
```

Now when making unauthenticated requests to our `/api` endpoints we will see some new headers in the response:
- `X-Rate-Limit-Limit` : `5`
- `X-Rate-Limit-Remaining` : `4`

After reaching the limit another header is added in the response, `X-Rate-Limit-Reset` which is the timestamp in seconds defining when rate limit will be reset and when api endpoint will be available again:
- `X-Rate-Limit-Limit` : `5`
- `X-Rate-Limit-Remaining` : `0`
- `X-Rate-Limit-Reset` : `1646998321`

An `ThrottleRequestsException` exception is raised and a response with status code `429: Too Many Requests` and content `Too many attempts` is returned.

### Customizing response

The response can be customized to provide different status code, content and headers. It can be done by adding a `get_response()` method to the limiter.

In the previous example it would look like this:

```python
class PremiumUsersLimiter(Limiter):
    # ...

    def get_response(self, request, response, headers):
        if request.user():
            return response.view("Too many attempts. Upgrade to premium account to remove limitations.", 400)
        else:
            return response.view("Too many attempts. Please try again tomorrow or create an account.", 400)
```
