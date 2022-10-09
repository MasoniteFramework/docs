CORS is sometimes important to activate in your application. CORS allows you to have a stronger layer of security over your application by specifying specific CORS headers in your responses. This is done through "preflight" requests.

These "preflight" requests are OPTIONS requests that are sent at the same time as other non safe requests (POST, PUT, DELETE) and specifically designed to verify the CORS headers before allowing the other request to go through.

To learn more about CORS please read [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

You can enable CORS protection by simply adding the CorsMiddleware into you middleware stack.

# Getting Started

To enable CORS in Masonite, you just need to add the CorsMiddleware in your middleware stack to your `http_middleware` key in your `Kernel.py` class.

**It is best to put the CORS middleware as the first middleware to prevent possible errors.**

```python
from masonite.middleware import CorsMiddleware

#..
class Kernel:

    http_middleware = [
        CorsMiddleware,
        EncryptCookies
    ]
    #..
```

Your application will now handle CORS requests to successfully go through.

# Configuration

All CORS settings are configured in the dedicated security configuration file `config/security.py`. The default
configuration options are:

```python
CORS = {
    "paths": ["api/*"],
    "allowed_methods": ["*"],
    "allowed_origins": ["*"],
    "allowed_headers": ["*"],
    "exposed_headers": [],
    "max_age": None,
    "supports_credentials": False,
}
```

## Paths

You can define paths for which you want CORS protection to be enabled. Multiple paths can be defined
in the list and wildcards (*) can be used to define the paths.

```python
    "paths": ["api/*", "auth/"]
```

Here all requests made to `auth/` and to all API routes will be protected with CORS.

## Allowed Methods

The default is to protect all HTTP methods but a list of methods can be specified instead. This will set
the `Access-Control-Allow-Methods` header in the response.

For example CORS can be enabled for sensitive requests:

```python
    "allowed_methods": ["POST", "PUT", "PATCH"]
```

## Allowed Origins

The default is to allow all origins to access a resource on the server. Instead you can define a list of origins
allowed to access the resources defined above (paths). Wildcards (*) can be used. This will set the `Access-Control-Allow-Origin` header in the response.

```python
    "allowed_origins": ["*.example.com"]
```

Here `blog.example.com` and `forum.example.com` will e.g. be authorized to make requests to the application paths defined above.

## Allowed Headers

The default is to authorized all request headers during a CORS request, but you can define a list of headers confirming that these are permitted headers to be used with the actual request. This will set the `Access-Control-Allow-Headers` header in the response.

```python
    "allowed_headers": ["X-Test-1", "X-Test-2"]
```

## Exposed Headers

The default is an empty list but you can define which headers will be accessible to the broswser e.g. with Javascript (with `getResponseHeader()`). This will set the `Access-Control-Expose-Headers` header in the response.

```python
    "exposed_headers": ["X-Client-Test-1"]
```

## Max Age

This will set the `Access-Control-Max-Age` header in the response and will indicates how long the results of a preflight request can be cached. The default is `None` meaning it preflight request results will never be cached.

You can indicate a cache duration in seconds:

```python
    "max_age": 3600
```

## Supports Credentials

This will set the `Access-Control-Allow-Credentials` and will indicate whether or not the response to the request can be exposed when the `credentials` flag is true. The default is `False`.

You can read more about it [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-allow-credentials).

```python
    "supports_credentials": True
```
