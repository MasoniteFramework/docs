# Validation

## Introduction

There are a lot of times when you need to validate incoming input either from a form or from an incoming json request. It is wise to have some form of backend validation as it will allow you. Masonite provides an extremely flexible and fluent way to validate this data.

# Validating The Request

Incoming form or JSON data can be validated very simply. All you  need to do is import the `Validator` class, resolve it, and use the necessary rule methods.

This whole snippet will look like this in your controller method:

```python
from masonite.validation import Validator

def show(self, request: Request, validate: Validator):
    """
    Incoming Input: {
        'user': 'username123',
        'email': 'user@example.com',
        'terms': 'on'
    }
    """
    valid = request.validate(

        validate.required(['user', 'email']),
        validate.accepted(['terms']])

    )

    if valid.errors:
        return request.back().with_errors()
```

This validating will read like "user and email are required and the terms must be accepted" (more on available rules and what they mean in a bit)

 