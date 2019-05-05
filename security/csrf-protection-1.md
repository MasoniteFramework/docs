# CSRF Protection

## Introduction

CSRF protection typically entails setting a unique token to the user for that page request that matches the same token on the server. This prevents any person from submitting a form without the correct token. There are many online resources that teach what CSRF does and how it works but Masonite makes it really simple to use.

## Getting Started

The CSRF features for Masonite are located in the `CsrfProvider` Service Provider and the `CsrfMiddleware`. If you do not wish to have CSRF protection then you can safely remove both of these.

The `CsrfProvider` simply loads the CSRF features into the container and the `CsrfMiddleware` is what actually generates the keys and checks if they are valid.

## Templates

By default, all `POST` requests require a CSRF token. We can simply add a CSRF token in our forms by adding the `{{ csrf_field }}` tag to our form like so:

```markup
<form action="/dashboard" method="POST">
    {{ csrf_field }}

    <input type="text" name="first_name">
</form>
```

This will add a hidden field that looks like:

```markup
<input type="hidden" name="__token" value="8389hdnajs8...">
```

If this token is changed or manipulated, Masonite will throw an `InvalidCsrfToken` exception from inside the middleware.

If you attempt a `POST` request without the `{{ csrf_field }}` then you will receive a `InvalidCsrfException` exception. This just means you are either missing the Jinja2 tag or you are missing that route from the `exempt` class attribute in your middleware.

## Exempting Routes

Not all routes may require CSRF protection such as OAuth authentication or various webhooks. In order to exempt routes from protection we can add it to the `exempt` class attribute in the middleware located at `app/http/middleware/CsrfMiddleware.py`:

```python
class CsrfMiddleware:
    """Verify CSRF Token Middleware
    """

    exempt = [
        '/oauth/github'
    ]

    ...
```

Now any POST routes that are to `your-domain.com/oauth/github` are not protected by CSRF and no checks will be made against this route. Use this sparingly as CSRF protection is crucial to application security but you may find that not all routes need it.

### Exempting Multiple Routes

You can also use `*` wildcards for exempting several routes under the same prefix. For example you may find yourself needing to do this:

```python
class CsrfMiddleware:
    """Verify CSRF Token Middleware
    """

    exempt = [
        '/api/document/reject-reason',
        '/api/document/@id/reject',
        '/api/document/@id/approve',
        '/api/document/@id/process/@user',
    ]

    ...
```

This can get a bit repetitive so you may specify a wildcard instead:

```python
class CsrfMiddleware:
    """Verify CSRF Token Middleware
    """

    exempt = [
        '/api/document/*',
    ]

    ...
```

