# Headers

## Introduction

Masonite allows you to easily add security headers to your application. Masonite adds some sensible defaults but you can modify them as you need.

## Configuration

All you need to do is add the middleware to your `HTTP_MIDDLEWARE` constant in your `config/middleware.py` file:

```python
from masonite.middleware import SecureHeadersMiddleware

HTTP_MIDDLEWARE = [
    ...
    SecureHeadersMiddleware,
]
```

This will add these default headers for your server:

```python
'Strict-Transport-Security': 'max-age=63072000; includeSubdomains'
'X-Frame-Options': 'SAMEORIGIN'
'X-XSS-Protection': '1; mode=block'
'X-Content-Type-Options': 'nosniff'
'Referrer-Policy': 'no-referrer, strict-origin-when-cross-origin'
'Cache-control': 'no-cache, no-store, must-revalidate'
'Pragma': 'no-cache'
```

## Overriding Headers

If you want to change or add any headers, you just need to specify them in your config/middleware.py file and this middleware will automatically pick them up. For example you can change the `X-Frame-Options` header like this:

{% code-tabs %}
{% code-tabs-item title="config/middleware.py" %}
```python
SECURE_HEADERS = {
   'X-Frame-Options' : 'deny'
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will then change your headers to:

```python
'Strict-Transport-Security': 'max-age=63072000; includeSubdomains'
'X-Frame-Options': 'deny'
'X-XSS-Protection': '1; mode=block'
'X-Content-Type-Options': 'nosniff'
'Referrer-Policy': 'no-referrer, strict-origin-when-cross-origin'
'Cache-control': 'no-cache, no-store, must-revalidate'
'Pragma': 'no-cache'
```

Notice the change in the new header we changed.

# CORS

You may also choose to use CORS for your application for advanced security measures. Using CORS is very similar to the secure headers above.

To get started just import the `CorsMiddleware` class into your `config/middleware.py` file and add it to your `HTTP_MIDDLEWARE` list:

```python
from masonite.middleware import CorsMiddleware
...
HTTP_MIDDLEWARE = [
    ...,
    CorsMiddleware,
]
```

Then below this list you can put your CORS headers as a dictionary. Here is a list of sensible defaults:

```python
from masonite.middleware import CorsMiddleware
...
HTTP_MIDDLEWARE = [
    ...,
    CorsMiddleware,
]

...

CORS = {
    'Access-Control-Allow-Origin': "*",
    "Access-Control-Allow-Methods": "DELETE, GET, HEAD, OPTIONS, PATCH, POST, PUT",
    "Access-Control-Allow-Headers": "Content-Type, Accept, X-Requested-With",
    "Access-Control-Max-Age": "3600",
    "Access-Control-Allow-Credentials": "true"
}
```

Now if you go to a browser you will see these headers being sent as a response from your server.