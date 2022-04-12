Masonite has several helpers available. Many helpers are used internally to help write clean Masonite code but can be used in your projects as well.

# Url

The Url helper allows you to create a full path URL:

```python
from masonite.helpers import url

url.url("/dashboard") #== example.com/dashboard
```

The URL will come from the `APP_URL` in your application config file.

You can also generate a URL for an asset:

```python
url.asset("s3.invoices", "invoice-01-2021.pdf")
```

You can also generate a URL for a route by its route name:

```python
url.route("users.profile", {"id": 1}) #== http://masonite.app/users/1/profile/
```

You can also generate just a path:

```python
url.route("users.profile", {"id": 1}, absolute=False) #== /users/1/profile/
```

# Compact

The compact helper is a shortcut helper when you want to compile a dictionary from variables.

There are times when you will have instances like this:

```python
from masonite.view import View

def show(self, view: View):
  users = User.all()
  articles = Article.all()
  return view.render('some.view', {"users": users, "articles": articles})
```

Notice we repeated the `users` and `articles` key the same as the variables name. In this case we can use the `compact` helper to clean the code up a bit:

```python
from masonite.view import View
from masonite.helpers import compact

def show(self, view: View):
  users = User.all()
  articles = Article.all()
  return view.render('some.view', compact(users, articles))
```

# Optional

The optional helper takes an object and allows any method calls on the object. If the method exists on the object it will return the value or else it will return None:

You may have a peice of code that looks like this:

```python
if request.user() and request.user().admin == 1:
  #.. do something
```

with the optional helper you can condense the code into something like this:

```python
from masonite.helpers import optional

if optional(request.user()).admin == 1:
  #.. do something
```

# Config

You can easily get configuration values using dot notation:

To get the application key for example:

```python
from masonite.configuration import config

key = config("application.key")
```

Event more convenient is the ability to use dot notation even inside a dictionary inside a configuration file.

Assuming you have a dictionary set inside a configuration like this:

```python
# config/appliation.py

HASHING = {
    "default": "bcrypt",
    "bcrypt": {"rounds": 10},
    "argon2": {"memory": 1024, "threads": 2, "time": 2},
}
```

You may use dot notation to get any value inside the `HASHING` constant like this:

```python
from masonite.configuration import config

key = config("application.hashing.bcrypt.rounds") #== 10
```

# Dump

You can easily dump variables into console for debugging, from inside a controller for example:
For this you can use Dump facade or the built-in `dump` python method:

```python
from masonite.facades import Dump

test = 1
data = {"key": "value"}
Dump.dump(test ,data)

# OR

dump(test, data)
```

This will dump data in console in a nice format to ease debugging.

# Dump and Die

If you want the code to stop and renders a dump page instead you can use the dump and die helper
named `dd`:

```python
from masonite.facades import Dump

test = 1
data = {"key": "value"}
Dump.dd(test ,data)

# OR

dd(test, data)
```

This will stop code at this line and renders a nice dump page where you can see all variables dumped
until now.

{% hint style="info" %}
Note that dumps will accumulate into session. If you want to clear dumps, you can use `Dump.clear()`
or you can enable the HTTP middleware `ClearDumpsBetweenRequestsMiddleware` to clear dumps
between every requests.
{% endhint %}

{% code title="Kernel.py" %}

```python
from masonite.middleware import ClearDumpsBetweenRequestsMiddleware

class Kernel:

    http_middleware = [
        #...
        ClearDumpsBetweenRequestsMiddleware
    ]
```

{% endcode %}
