Masonite has several helpers available. Many helpers are used internally to help write clean Masonite code but can be used in your projects as well.

All helpers can be imported from `helpers` module.

# Masonite Specifics

## app

Get easily access to [application container](../architecture/service-container.md):

```python
from masonite.helpers import app

app().make("session")
```

You can also resolve dependencies directly and even pass arguments when resolving:

```python
from masonite.helpers import app

app("session") #== Session

app("my_service", args)
```

## dump

You can easily dump variables into console for debugging, from inside a controller for example:
For this you can use Dump facade or the built-in `dump` python method:

```python
from masonite.facades import Dump

test = 1
data = {"key": "value"}
Dump.dump(test, data)

# OR

dump(test, data)
```

This will dump data in console in a nice format to ease debugging.

## dd

If you want the code to stop and renders a dump page instead you can use the dump and die helper
named `dd`:

```python
from masonite.facades import Dump

test = 1
data = {"key": "value"}
Dump.dd(test, data)

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

## config
TODO

## env
TODO


# Paths

## base_path

Get the absolute path to your project root directory or build the absolute path to a given file relative to the project root directory.
```python
from masonite.utils.location import base_path

base_path() # /Users/johndoe/my-project/

base_path("storage/framework") # /Users/johndoe/my-project/storage/framework
```

## views_path

Get the absolute path to your project views directory or build the absolute path to a given file relative to the project views directory.

```python
from masonite.utils.location import views_path

views_path()  # /Users/johndoe/my-project/templates

views_path("admin/index.html")  # /Users/johndoe/my-project/templates/admin/index.html
```


## controllers_path

Get the absolute path to your project controllers directory or build the absolute path to a given file relative to the project controllers directory.

```python
from masonite.utils.location import controllers_path

controllers_path()  # /Users/johndoe/my-project/app/controllers

controllers_path("admin/AdminController.py")  # /Users/johndoe/my-project/app/controllers/admin/AdminController.py
```

## mailables_path

Get the absolute path to your project mailables directory or build the absolute path to a given file relative to the project mailables directory.

```python
from masonite.utils.location import mailables_path

mailables_path()  # /Users/johndoe/my-project/app/mailables

mailables_path("WelcomeUser.py")  # /Users/johndoe/my-project/app/mailables/WelcomeUser.py
```

## config_path

Get the absolute path to your project config directory or build the absolute path to a given file relative to the project config directory.

```python
from masonite.utils.location import config_path

config_path()  # /Users/johndoe/my-project/config

config_path("custom.py")  # /Users/johndoe/my-project/config/custom.py
```


## migrations_path

Get the absolute path to your project migrations directory or build the absolute path to a given file relative to the project migrations directory.

```python
from masonite.utils.location import migrations_path

migrations_path()  # /Users/johndoe/my-project/databases/migrations

migrations_path("2022_11_01_043202_create_users_table.py")  # /Users/johndoe/my-project/databases/migrations/2022_11_01_043202_create_users_table.py
```

## seeds_path

Get the absolute path to your project seeds directory or build the absolute path to a given file relative to the project seeds directory.

```python
from masonite.utils.location import seeds_path

seeds_path()  # /Users/johndoe/my-project/databases/seeds

seeds_path("products_table_seeder.py")  # /Users/johndoe/my-project/databases/seeds/products_table_seeder.py
```

## jobs_path

Get the absolute path to your project jobs directory or build the absolute path to a given file relative to the project jobs directory.

```python
from masonite.utils.location import jobs_path

jobs_path()  # /Users/johndoe/my-project/app/jobs

jobs_path("SendInvoices.py")  # /Users/johndoe/my-project/app/jobs/SendInvoices.py
```

## resources_path

Get the absolute path to your project resources directory or build the absolute path to a given file relative to the project resources directory.

```python
from masonite.utils.location import resources_path

resources_path()  # /Users/johndoe/my-project/resources

resources_path("css/app.css")  # /Users/johndoe/my-project/resources/css/app.css
```

## models_path

Get the absolute path to your project models directory or build the absolute path to a given file relative to the project models directory.

```python
from masonite.utils.location import models_path

models_path()  # /Users/johndoe/my-project/app/models

models_path("Product.py")  # /Users/johndoe/my-project/app/models/Product.py
```

{% hint style="info" %}
All above paths helper return an absolute path to the location. When providing a file path to the helper you can set `absolute=False` to get the
path relative given directory.
{% endhint %}

```python
from masonite.utils.location import models_path

models_path("blog/Article.py", absolute=False)  # app/models/blog/Article.py
```

# URLs and Routes

## Url
The Url helper allows you to create a full path URL:

```python
from masonite.helpers import url

url.url("/dashboard") #== example.com/dashboard
```

The URL will come from the `APP_URL` in your application config file.

It accepts a dictionary to add query string parameters when building the url:

```python
url.url("/search/users",  query_params={"search": "John Doe" "order": "asc"})
#== http://masonite.app/users/?search=John%2Doe&order=asc
```

## Asset
You can also generate a URL for an asset:

```python
url.asset("s3.invoices", "invoice-01-2021.pdf")
```

## Route
You can also generate a URL for a route by its route name:

```python
url.route("users.profile", {"id": 1}) #== http://masonite.app/users/1/profile/
```

You can also generate just a path:

```python
url.route("users.profile", {"id": 1}, absolute=False) #== /users/1/profile/
```

It accepts a dictionary to add query string parameters when building the route url:

```python
url.route("user.profile", {"id": 1}, query_params={"preview": 1})
#== http://masonite.app/users/1/profile/?preview=1
```

# Python Helpers

## compact

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

## optional

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

## collect
TODO

## flatten
TODO
