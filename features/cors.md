CORS is sometimes important to activate in your application. CORS allows you to have a stronger layer of security over your application by specifying specific CORS headers in your responses. This is done through "preflight" requests. 

These "preflight" requests are OPTIONS requests that are sent at the same time as other non safe requests (POST, PUT, DELETE) an specifically designed to verify the CORS headers before allowing the other request to go through.

> If your application receives a preflight requests without having CORS enabled then you will get an error in the response.

# Getting Started

To enable cors in Masonite, we simply need to create a CORSMiddleware file inside your project, inherit from Masonite's class, set your headers and add it to the HTTP middleware inside your Kernel.

So first create your middleware class wherever you put your middleware. Here's an example class:

```python
from masonite.middleware import Middleware
from masonite.middleware.route import CorsMiddleware

class CorsMiddleware(CorsMiddleware):

    headers = {
        'Access-Control-Allow-Origin': "*",
        "Access-Control-Allow-Methods": "DELETE, GET, HEAD, OPTIONS, PATCH, POST, PUT",
        "Access-Control-Allow-Headers": "Content-Type, Accept, X-Requested-With",
        "Access-Control-Max-Age": "3600",
        "Access-Control-Allow-Credentials": "true"
    }
```

You may add any valid CORS header. You can learn more about CORS headers in the [Mozilla Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

Once created you can add it to your `http_middleware` key in your `Kernel.py` class. 

**It is best to put the CORS middleware as the first middleware to prevent possible errors.**

```python
from app.middleware.CorsMiddleware import CorsMiddleware

#..
class Kernel:

    http_middleware = [
        CorsMiddleware,
        EncryptCookies
    ]
    #..
```

Your application will now allow CORS requests to successfully go through.