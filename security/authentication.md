# Authentication

## Introduction

Masonite comes with some authentication out of the box but leaves it up to the developer to implement. Everything is already configured for you by default. The default authentication model is the `app/User` model but you can change this in the `config/auth.py` configuration file.

## Configuration

There is only a single `config/auth.py` configuration file which you can use to set the authentication behavior of your Masonite project. If you wish to change the authentication model, to a `app/Company` model for example, feel free to do in this configuration file.

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

If you want to authenticate a model, you can use the `Auth` facade that ships with Masonite. This is simply a class that is used to authenticate models with a `.login()` method.

In order to authenticate a model this will look like:

```python
from masonite.facades.Auth import Auth

def show(self, Request):
    Auth(Request).login('user@email.com', 'password')
```

This will find a model with the supplied username, check if the password matches using `bcrypt` and return the model. If it is not found or the password does not match, it will return `False`.

{% hint style="warning" %}
Again all authenticating models need to have a `password` column. The column being used to authenticate, such as a username or email field can be specified in the model using the `__auth__` class attribute.
{% endhint %}

### Changing The Authentication Field

You may change the column to be authenticated by simply changing the column value of the `__auth__` class attribute. This will look something like:

```python
class User(Model):

    __fillable__ = ['name', 'email', 'password']

    __auth__ = 'email'
```

This will look inside the `email` column now and check that column and password. The authentication column is `email` by default.

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
def show(self, Request):
    Request.user()
```

If you wish not to use middleware to load the user into the request you can get the request by again using the `Auth` class

```python
from masonite.facades.Auth import Auth

def show(self, Request):
    Auth(Request).user()
```

### Checking if the User is Authenticated

If you would like to simply check if the user is authenticated, `Request.user()` or `Auth(Request).user()` will return `False` if the user is not authenticated. This will look like:

```python
def show(self, Request):
    if Request.user():
        user_email = Request.user().email
```

### Logging In By ID

Sometimes in your application you may want to sign in a user without the need for a username and password. This can be used on the administrative end of your application to login to other user's accounts to verify information.

```python
def show(self, Auth, Request):
    Auth(Request).login_by_id(1):
```

This will login the user with the ID of 1 and skip over the username and password verification completely.

### Logging In Once

All logging in methods will set a cookie in the user's browser which will then be verified on subsequent requests. Sometimes you may just want to login the user "once" which will authenticate the user but not set any cookies. We can use the `once()` method for this. This method can be used with all the previous methods by calling it first

```python
def show(self, Auth, Request):
    Auth(Request).once().login_by_id(1)
    # or 
    Auth(Request).once().login('user@email.com', 'secret')
```

This will return an instance of the user model just authenticated or it will return `False` if the authentication failed. This method will not save state.

{% hint style="info" %}
This method is great for creating a stateless API or when you just need to verify a user with a username and password
{% endhint %}

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
def logout(self, Request):
    Auth(Request).logout()
    return Request.redirect('/login')
```

Great! You’ve mastered how Masonite uses authentication. Remember that this is just out of the box functionality and you can create a completely different authentication system but this will suffice for most applications.

