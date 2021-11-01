Masonite makes authentication really simply.

# Configuration

The configuration for Masonite's authentication is quite simple:

```python
from app.User import User

GUARDS = {
    "default": "web",
    "web": {"model": User},
    "password_reset_table": "password_resets",
    "password_reset_expiration": 1440,  # in minutes. 24 hours. None if disabled
}

```

The default key here is the guard to use for authentication. The `web` dictionary is the configuration for the web guard.

# Login Attempts

You can attempt a login by using the `Auth` class and using the `attempt` method:

```python
from masonite.authentication import Auth
from masonite.request import Request

def login(self, auth: Auth, request: Request):
  user = auth.attempt(request.input('email'), request.input("password"))
```

If the attempt succeeds, the user will now be authenticated and the result of the attempt will be the authenticated model.

If the attempt fails then the result will be `None`.

If you know the primary key of the model, you can attempt by the id:

```python
from masonite.authentication import Auth
from masonite.request import Request

def login(self, auth: Auth, request: Request):
  user = auth.attempt_by_id(1)
```

You can logout the current user:

```python
from masonite.authentication import Auth
from masonite.request import Request

def logout(self, auth: Auth):
  user = auth.logout()
```

# User

You can get the current authenticated user:

```python
from masonite.authentication import Auth
from masonite.request import Request

def login(self, auth: Auth, request: Request):
  user = auth.user() #== <app.User.User>
```

If the user is not authenticated, this will return `None`.

# Routes

You can register several routes quickly using the auth class:

```python
from masonite.authentication import Auth

ROUTES = [
  #..
]

ROUTES += Auth.routes() 
```

This will register the following routes:

| URI                   | Description                                             |
| --------------------- | ------------------------------------------------------- |
| GET /login            | Displays a login form for the user                      |
| POST /login           | Attempts a login for the user                           |
| GET /home             | A home page for the user after a login attempt succeeds |
| GET /register         | Displays a registration form for the user               |
| POST /register        | Saved the posted information and creates a new user     |
| GET /password_reset   | Displays a password reset form                          |
| POST /password_reset  | Attempts to reset the users password                    |
| GET /change_password  | Displays a form to request a new password               |
| POST /change_password | Requests a new password                                 |

# Guards

Guards are encapsulated logic for logging in, registering and fetching users. The web guard uses a `cookie` driver which sets a `token` cookie which is used later to fetch the user.

You can switch the guard on the fly to attempt authentication on different guards:

```python
from masonite.authentication import Auth
from masonite.request import Request

def login(self, auth: Auth, request: Request):
  user = auth.guard("custom").attempt(request.input('email'), request.input("password")) #== <app.User.User>
```

# Gates & Policies

TBD