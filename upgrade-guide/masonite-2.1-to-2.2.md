# Masonite 2.1 to 2.2

# Introduction

Welcome to the upgrade guide to get your Masonite 2.1 application working with Masonite 2.2. We'll be focusing on all the breaking changes so we can get all your code working on a Masonite 2.2 release cycle.

We will not go into all the better ways to use some of the features. For those changes be sure to read the "Whats New in 2.1" documentation to the left to see what fits into your application and what doesn't.

Masonite 2.2 is jam packed with amazing new features and most of which are backwards compatible so upgrading from Masonite 2.1 to 2.2 is really simple.

We'll go through each section that your application will need to be upgraded and how it can be done.

# Getting Started

First let's upgrade Masonite 2.2 first so we can see any exceptions that will be raised.

Let's upgrade by doing:

```
pip install masonite==2.2.0
```

# Removing route helpers

In Masonite 2.1, route helpers were deprecated and you likely started receiving deprecation warnings. In Masonite 2.2, these were removed. You may have had routes that looks like this:

```python
from masonite.helpers.routes import get, post

ROUTES = [
    get('/url/home').name('home')
]
```

You will now need to remove all these and use the class based ones. To make this easier we can just import the get and post helpers and alias them like this:

```python
from masonite.routes import Get as get, Post as post

ROUTES = [
    get('/url/home').name('home')
]
```

# Changed Validation

## Validation Provider

You'll need to add a new validation provider if you want your application to have the new validation features.

Add it by importing it into `config/providers.py` and add it to your `PROVIDERS` list:

```python
from masonite.validation.providers import ValidationProvider

PROVIDERS = [
    ...
    ValidationProvider,
    ...
]
```

## Validation Code

Masonite 2.2 completely removed the validation package from 2.1 and created an even better all new validation package. You'll have to remove all your validation classes and use the new validation package.

For example you may have had a validation class that looked like this:

```python
class RegisterValidator(Validator):

    def register(self):
        users = User.all()
        self.messages({
            'email': 'That email already exists',
            'username': 'Usernames should be between 3 and 20 characters long'
        })

        return self.validate({
            'username': [Required, Length(3, 20)],
            'email': [Required, Length(1), Not(In(users.pluck('email')))],
            'password': [Required]
        })
```

and used it inside your views like this:

```python
from app.validators import RegisterValidator

    def store(self):
        validate = RegisterValidator(self.request).register()
        if validate.check():
            validate.check_exists()

        if not validate.check():
            self.request.session.flash('validation', json.dumps(validate.errors()))
            return self.request.redirect_to('register')
```

This is now completely changed to use a better and more sleeker validation. The above validation can now be written like this:

```python
from masonite.validation import Validator

    def store(self, request: Request, validator: Validator):
        errors = request.validate(
            validator.required(['username', 'email', 'password'])
            validator.length('username', min=3, max=20)
            validator.length('email', min=3)
            validator.isnt(
                validator.is_in('email', User.all().pluck('email'))
            )
        )

        if errors:
            return self.request.redirect_to('register').with_errors(errors)
```

You can do a lot of other awesome things like rule enclosures. Read more under the Validation documentation

## Auth class now auto resolves it's own request class

Masonite 2.2 changes a bit how the `masonite.auth.Auth` class resolves out of the container and how it resolves its own dependencies.

Now instead of doing something like:

```python
from masonite.auth import Auth
...
def show(self, request: Request):
    Auth(request).user()
```

You'll need to move this into the parameter list so it can be resolved:

```python
from masonite.auth import Auth
...
def show(self, auth: Auth):
    auth.user()
```

There should be quite a bit of these in your application if you have used this class or you have used the built in `craft auth` scaffold command.

Here is an example application that is being upgraded from 2.1 to 2.2 [GitHub Repo](https://github.com/josephmancuso/gbaleague-masonite2/pull/2/files)


## Resolving Classes

The behavior for resolving classes has now been changes. If you bind a class into the container like this:

```
from some.place import SomeClass

self.app.bind('SomeClass', SomeClass)
```

This will now resolve from the container *when you resolve it as a parameter list*. This means that you will never get back a class inside places like controllers. Before you may have gotten it back from the container and passed in dependencies yourself like:

```python
from some.place import SomeClass

def show(self, request: Request, some: SomeClass):
    setup = some(request)
```

But this is no longer possible. You will now only get back objects so you will not be able to instantiate these variables. So take this into account if you have bound classes into the container.