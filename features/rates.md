# Rate Limiting

Masonite includes a rate limiting feature which make it really easy to limit an action during a given time window.

The most frequent use of this feature is to add throttling to some API endpoints but you could also
use this to limit how many time a model can be updated or a job can be run in the queue.





## Overview


## Existing limits

## Custom limits


## Add Rate Limits to HTTP Requests

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
This middleware is taking one argument which can be either a limit string or a limiter name. Here
we will use a limiter.

Create and register a [Limiter](#limiters) in your application provider that will control the throttling:

```python
from masonite.facades import RateLimiter
from masonite.rates import GuestsOnlyLimiter


class AppProvider(Provider):

    def register(self):
        RateLimiter.register("api", GuestsOnlyLimiter("5/hour"))
```
Here we have used an existing limiter which limit only non authenticated user to 100 actions per day under the `api` name.

We now just have to use it in our routes:
```python
# web.py
Route.group(
    [
      Route.post("/uploads/", "UploadController@create")
    ],
    middleware=["throttle:api"],
    prefix="/api",
)
```
`api` is the limiter name given as argument to our throttle middleware.

Now when making unauthenticated requests to our `/api` endpoints we will see some new headers in the response:
- `X-Rate-Limit-Limit` : `5`
- `X-Rate-Limit-Remaining` : `4`

After reaching the limit another header is added in the response, `X-Rate-Limit-Reset` which is the timestamp in seconds defining when rate limit will be reset and when api endpoint will be available again:
- `X-Rate-Limit-Limit` : `5`
- `X-Rate-Limit-Remaining` : `0`
- `X-Rate-Limit-Reset` : `1646998321`

We also get a response with status `429: Too Many Requests` and with content `Too many attempts`.

### Customizing response

The response can be customized to provide a different status code or content:

## Add Rate Limits to any action

### callable ?

### callable