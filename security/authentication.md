# Authentication

## Introduction

Masonite comes with some authentication out of the box but leaves it up to the developer to implement. Everything is already configured for you by default. The default authentication model is the `app/User` model but you can change this in the `config/auth.py` configuration file.

## Configuration

There is only a single `config/auth.py` configuration file which you can use to set the authentication behavior of your Masonite project. If you wish to change the authentication model, to a `Company` model for example, feel free to do in this configuration file.

## Authentication Model

Again the default authentication model is the `app/User` model which out of the box comes with a `__auth__` class attribute. This attribute should be set to the column that you want to authenticate with. By default your `app/User` model will default to the `email` column but if you wish to change this to another column such as `name`, you can do so here. This will lead your model to look like:

```python
class User(Model):

    __fillable__ = ['name', 'email', 'password']

    __auth__ = 'name'
```

{% hint style="info" %}
**All models that should be authenticated in addition to specifying a** `__auth__` **attribute also needs to have a** `password` **field as well in order to use the out of the box authentication that comes with Masonite.**
{% endhint %}

## Authenticating a Model

If you want to authenticate a model, you can use the `Auth` class that ships with Masonite. This is simply a class that is used to authenticate models with a `.login()` method.

In order to authenticate a model this will look like:

```python
from masonite.auth import Auth

def show(self, request: Request):
    Auth(request).login('user@email.com', 'password')
```

This will find a model with the supplied username, check if the password matches using `bcrypt` and return the model. If it is not found or the password does not match, it will return `False`.

{% hint style="warning" %}
Again all authenticating models need to have a `password` column. The column being used to authenticate, such as a username or email field can be specified in the model using the `__auth__` class attribute.
{% endhint %}

### Changing The Authentication Model

You may change the column to be authenticated by simply changing the column value of the `__auth__` class attribute. This will look something like:

```python
class User(Model):

    __fillable__ = ['name', 'email', 'password']

    __auth__ = 'email'
```

This will look inside the `email` column now and check that column and password. The authentication column is `email` by default.

### Multiple Authentication Columns

Sometimes you will want to authenticate using either the username or the email. You can set the columns to authenticate against as a list:

```python
class User(Model):

    __fillable__ = ['name', 'email', 'password']

    __auth__ = ['email', 'name']
```

This will now authenticate using the password supplied and EITHER the `username` OR the `email` if either one matches.

### Changing The Authentication Password

When given a password, Masonite will use the `password` column to authenticate. This can be changed by specifying a `__password__` attribute:

```python
class User(Model):

    __fillable__ = ['name', 'email', 'user_passwords']

    __auth__ = 'email'
    __password__ = 'user_passwords'
```

## Creating an Authentication System

You may of course feel free to roll your own authentication system if you so choose but Masonite comes with one out of the box but left out by default. In order to scaffold this authentication system you can of course use a `craft` command:

```text
$ craft auth
```

This will create some controllers, views and routes for you. This command should be used primarily on fresh installs of Masonite but as long as the controllers do not have the same names as the controllers being scaffolded, you will not have any issues.

The views scaffolded will be located under `resources/templates/auth`.

After you have ran the `craft auth` command, just run the server and navigate to `http://localhost:8000/login` and you will now have a login, registration and dashboard. Pretty cool, huh?

### Retrieving the Authenticated User

Masonite ships with a `LoadUser` middleware that will load the user into the request if they are authenticated. Masonite uses the `token` cookie in order to retrieve the user using the `remember_token`column in the table.

Using this `LoadUser` middleware you can retrieve the current user using:

```python
def show(self, request: Request):
    request.user()
```

If you wish not to use middleware to load the user into the request you can get the request by again using the `Auth` class

```python
from masonite.auth import Auth

def show(self, request: Request):
    Auth(request).user()
```

### Checking if the User is Authenticated

If you would like to simply check if the user is authenticated, `request.user()` or `Auth(request).user()` will return `False` if the user is not authenticated. This will look like:

```python
def show(self, request: Request):
    if request.user():
        user_email = request.user().email
```

## Logging In

You can easily log users into your application using the Auth class which takes the request object as a dependency:

```python
from masonite.auth import Auth

def show(self, request: Request):
    Auth(request).login(
        request.input('username'),
        request.input('password')
    )
```

{% hint style="info" %}
Note that the username you supply needs to be in whatever format the `__auth__` attribute is on your model. If the email address is the "username", then the user will need to supply their email address.
{% endhint %}

### Login By ID

If you need more direct control internally, you can login by the models ID:

```python
from masonite.auth import Auth

def show(self):
    Auth(request).login_by_id(1)
```

You are now logged in as the user with the ID of 1.

### Login Once

If you only want to login "once", maybe for just authenticating an action or verifying the user can supply the correct credentials, you can login without saving any cookies to the browser:

```python
from masonite.auth import Auth

def show(self):
    Auth(request).once().login_by_id(1)
```

You can do the same for the normal login method as well:

```python
from masonite.auth import Auth

def show(self, request: Request):
    Auth(request).once().login(
        request.input('username'),
        request.input('password')
    )
```

## Protecting Routes

Masonite ships with an authentication middleware. You can use this middleware as a route middleware to protect certain routes from non authenticated users. This is great for redirecting users to a login page if they attempt to go to their dashboard.

You can use this middleware in your routes file like so:

```python
Get().route('/dashboard', 'DashboardController@show').middleware('auth')
```

By default this will redirect to the route named `login`. If you wish to redirect the user to another route or to a different URI, you can edit the middleware in `app/http/middleware/AuthenticationMiddleware.py`

### Logging Out a User

If you wish to end the session for the user and log them out, you can do so by using the `Auth` class. This looks like:

```python
Auth(request).logout()
```

This will delete the cookie that was set when logging in. This will not redirect the user to where they need to go. A complete logout view might look like:

```python
def logout(self, request: Request):
    Auth(request).logout()
    return request.redirect('/login')
```

## Verifying A User's Email

If you wish to require a user to verify their email address and automatically send them an email, you can extend the `User` model.

```python
from masonite.auth import MustVerifyEmail

class User(Model, MustVerifyEmail):

    __fillable__ = ['name', 'email', 'password']

    __auth__ = 'name'
```

When a user registers this will automatically send them an email asking them to confirm their email address.

### Redirecting Unverified User's

You can use the `VerifyEmailMiddleware` class to redirect an unverified user.

You can use this middleware in your routes file like so:

```python
Get().route('/dashboard', 'DashboardController@show').middleware('verified')
```

Great! You’ve mastered how Masonite uses authentication. Remember that this is just out of the box functionality and you can create a completely different authentication system but this will suffice for most applications.

