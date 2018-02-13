# Authentication
# Introduction
Masonite comes with some authentication out of the box but leaves it up to the developer to implement. Everything is already configured for you by default. The default authentication model is the `app/User` model but you can change this in the `config/auth.py` configuration file.

## Configuration
There is only a single `config/auth.py` configuration file which you can use to set the authentication behavior of your Masonite project. If you wish to change the authentication model, to a `app/Company` model for example, feel free to do in this configuration file.

## Authentication Model
Again the default authentication model is the `app/User` model which out of the box comes with a `__auth__` class attribute. This attribute should be set to the column that you want to authenticate with. By default your `app/User` model will default to the `email` column but if you wish to change this to another column such as `name`, you can do so here. This will lead your  model to look like:

```python
class User(Model):

    __fillable__ = ['name', 'email', 'password']

    __auth__ = 'name'
```

All models that should be authenticated in addition to specifying a `__auth__` attribute also needs to have a `password` field as well in order to use the out of the box authentication that comes with Masonite.

## Authenticating a Model
If you want to authenticate a model, you can use the `Auth` facade that ships with Masonite. This is simply a class that is used to authenticate models with a `.login()` method.

In order to authenticate a model this will look like:

```python
from masonite.facades.Auth import Auth

Auth.login('user@email.com', 'password')
```

This will find a model with the supplied username, check if the password matches using `bcrypt` and return the model. If it is not found or the password does not match, it will return `False`.

Again all authenticating models need to have a `password` column. The column being used to authenticate, such as a username or email field can be specified in the model using the `__auth__` class attribute.

## Creating an Authentication System
You may of course feel free to roll your own authentication system if you so choose but Masonite comes with one out of the box but left out by default. In order to scaffold this authentications system you can of course use a `craft` command:

    $ craft auth

This will create some controllers, views and routes for you. This command should be used primarily on fresh installs of Masonite but as long as the controllers do not have the same names as the controllers being scaffolded, you will not have any issues.

The views scaffolded will be located under `resources/templates/auth`.

After you have ran the `craft auth` command, just run the server and navigate to `http://localhost:8000/login` and you will now have a login, registration and dashboard. Pretty cool, huh?

## Retrieving the Authenticated User
Masonite ships with a `LoadUser` middleware that will load the user into the request if they are authenticated. Masonite uses the `token` cookie in order to retrieve the user using the `remember_token`column in the table.

Using this `LoadUser` middleware you can retrieve the current user using:

```python
def show(self, request):
    request.user()
```

If you wish not to use middleware to load the user into the request you can get the request by again using the `Auth` class

```python
from masonite.facades.Auth import Auth

def show(self, request):
    Auth(request).user()
```

### Checking if the User is Authenticated

If you would like to simply check if the user is authenticated, `request.user()` or `Auth(request).user()` will return `False` if the user is not authenticated. This will look like:

```python
def show(self, request):
    if request.user():
        user_email = request.user().email
```

## Protecting Routes
Masonite ships with an authentication middleware. You can use this middleware as a route middleware to protect certain routes from non authenticated users. This is great for redirecting users to a login page if they attempt to go to their dashboard.

You can use this middleware in your routes file like so:

```python
Get().route('/dashboard', 'DashboardController@show').middleware('auth')
```

By default this will redirect to the route named `login`. If you wish to redirect the user to another route or to a different URI, you can edit the middleware in `app/http/middleware/AuthenticationMiddleware.py`

## Logging Out a User
If you wish to end the session for the user and log them out, you can do so by using the `Auth` class. This looks like:

```python
Auth(request).logout()
```

This will delete the cookie that was set when logging in. This will not redirect the user to where they need to go. A complete logout view might look like:

```python
def logout(self, request):
        Auth(request).logout()
        return request.redirect('/login')
```

Great! You’ve mastered how Masonite uses authentication. Remember that this is just out of the box functionality and you can create a completely different authentication system but this will suffice for most applications.






