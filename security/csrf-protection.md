# CSRF Protection

## Introduction

Masonite 1.4+ now has out of the box CSRF protection. CSRF, or Cross-Site Request Forgery is when malicious actors attempt to send requests \(primarily POST requests\) on your behalf. CSRF protection typically entails setting a unique token to the user for that page request that matches the same token on the server. This prevents any person from submitting a form without the correct token. There are many online resources that teach what CSRF does and how it works but Masonite makes it really simple to use.

If you are using Masonite 1.4 already then you already have the correct middleware and Service Providers needed. You can check which version of Masonite you are using by simply running `pip show masonite` and looking at the version number.

## Getting Started

{% hint style="info" %}
If you are running a version of Masonite before 1.4 then check the upgrade guide for [Masonite 1.3 to 1.4](../upgrade-guide/masonite-1.3-to-1.4.md) for learning how to upgrade.
{% endhint %}

## Usage

The CSRF features for Masonite are located in the `CsrfProvider` Service Provider and the `CsrfMiddleware`. If you do not wish to have CSRF protection then you can safely remove both of these.

The `CsrfProvider` simply loads the CSRF features into the container and the `CsrfMiddleware` is what actually generates the keys and checks if they are valid.

### Templates

By default, all `POST` requests require a CSRF token. We can simply add a CSRF token in our forms by adding the `{{ csrf_field|safe }}` tag to our form like so:

```markup
<form action="/dashboard" method="POST">
    {{ csrf_field|safe }}

    <input type="text" name="first_name">
</form>
```

This will add a hidden field that looks like:

```markup
<input type="hidden" name="csrf_token" value="8389hdnajs8...">
```

If this token is changed or manipulated, Masonite will throw an `InvalidCsrfToken` exception from inside the middleware.

If you attempt a `POST` request without the `{{ csrf_field|safe }}` then you will receive a `KeyError: 'csrf_token'` exception. This just means you are either missing the Jinja2 tag or you are missing that route from the `exempt` class attribute in your middleware.

### Exempting Routes

Not all routes may require CSRF protection such as OAuth authentication. In order to exempt routes from protection we can add it to the `exempt` class attribute in the middleware located at `app/http/middleware/CsrfMiddleware.py`:

```python
class CsrfMiddleware:
    ''' Verify CSRF Token Middleware '''

    exempt = [
        '/oauth/github'
    ]

    ...
```

Now any POST routes that are to `your-domain.com/oauth/github` are not protected by CSRF. Use this sparingly as CSRF protection is crucial to application security but you may find that not all routes need it.

