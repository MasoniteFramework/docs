# Views

## Introduction

Views contain all the HTML that you’re application will use to render to the user. Unlike Django, views in Masonite are your HTML templates. All views are located inside `resources/templates`directory.

All views are rendered with Jinja2 so we can use all the Jinja2 code you are used to. An example view looks like:

{% code-tabs %}
{% code-tabs-item title="resources/templates/helloworld.html" %}
```markup
<html>
  <body>
    <h1> Hello {{ 'world' }}</h1>
  </body>
</html>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Creating Views

Since all views are located in `resources/templates`, we can use simply create all of our views manually here or use our `craft` command tool. To make a view just run:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft view hello
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create a template under `resources/templates/hello.html`.

## Calling Views

### Helper Function

There are several ways we can call views in our controllers. The first recommended way is using the `view()` function. Masonite ships with a `HelpersProvider` Service Provider. This provider will add several new built in functions to your project. These helper functions can be used as shorthand for several commonly used classes such as the `View` and `Request` class. See the [Helper Functions](helper-functions.md) documentation for more information.

One of the helper functions is the `view()` function which is accessible like any other built in Python function.

We can call views in our controllers like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self):
    return view('dashboard')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will return the view located at `resources/templates/dashboard.html`. We can also specify a deeper folder structure like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self):
    return view('profiles/dashboard')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will look for the view at `resources/templates/profiles/dashboard.html`

### From The Container

The `View` class is loaded into the container so we can retrieve it in our controller methods like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self, View):
    return View('dashboard')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This is exactly the same as using the helper function above. So if you choose to code more explicitly, the option is there for you.

{% hint style="success" %}
If this looks weird to you or you are not sure how the container integrates with Masonite, make sure you read the [Service Container](../architectural-concepts/service-container.md) documentation
{% endhint %}

### Global Views

Some views may not reside in the `resources/templates` directory and may even reside in third party packages such as a dashboard package. We can locate these views by passing a `/` in front of our view.

For example as a use case we might pip install a package:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ pip install package-dashboard
```
{% endcode-tabs-item %}
{% endcode-tabs %}

and then be directed or required to return one of their views:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self):
    return view('/package/views/dashboard')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will look inside the `dashboard.views` package for a `dashboard.html` file and return that. You can obviously pass in data as usual.

#### Caveats

It's important to note that if you are building a third party package that integrates with Masonite that you place any `.html` files inside a Python package instead of directly inside a module. For example, you should place .html files inside a file structure that looks like:

```python
package/
  views/
    __init__.py
    index.html
    dashboard.html
setup.py
MANIFEST.in
...
```

and not inside the package directory. This is a Jinja limitation that says that all templates should be located in packages.

Accessing a global view such as:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self):
    return view('/package/dashboard')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

will perform a relative import for your Masonite project. For example it will catch:

```text
app/
config/
databases/
...
package/
  dashboard.html
```

So if you are making a package for Masonite then keep this in mind in where you should put your templates

## Passing Data to Views

A lot of the time we’ll need to pass in data to our views. This data is passed in with a dictionary that contains a key which is the variable with the corresponding value. We can pass data to the function like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self, request: Request):
    return view('dashboard', {'id': Request.param('id')})
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
Remember that by passing in parameters like `Request` to the controller method, we can retrieve objects from the IOC container. Read more about the IOC container in the [Service Container](../architectural-concepts/service-container.md) documentation.
{% endhint %}

This will send a variable named `id` to the view which can then be rendered like:

{% code-tabs %}
{% code-tabs-item title="resources/templates/dashboard.html" %}
```markup
<html>
  <body>
    <h1> {{ id }} </h1>
  </body>
</html>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Adding Environments

{% hint style="success" %}
This section requires knowledge of [Service Providers](../architectural-concepts/service-providers.md) and how the [Service Container](../architectural-concepts/service-container.md) works. Be sure to read those documentation articles.
{% endhint %}

You can also add Jinja2 environments to the container which will be available for use in your views. This is typically done for third party packages such as Masonite Dashboard. You can extend views in a Service Provider in the boot method. Make sure the Service Provider has the `wsgi` attribute set to `False`. This way the specific Service Provider will not keep adding the environment on every request.

```python
wsgi = False

...

def boot(self, ViewClass):
    ViewClass.add_environment('dashboard/templates')
```

By default the environment will be added using the PackageLoader Jinja2 loader but you can explicitly set which loader you want to use:

```python
from jinja2 import FileSystemLoader
...
wsgi = False

...

def boot(self, ViewClass):
    ViewClass.add_environment('dashboard/templates', loader=FileSystemLoader)
```

The default loader of PackageLoader will work for most cases but if it doesn't work for your use case, you may need to change the loader.

