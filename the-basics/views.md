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

## Passing Data to Views

A lot of the time we’ll need to pass in data to our views. This data is passed in with a dictionary that contains a key which is the variable with the corresponding value. We can pass data to the function like so:

{% code-tabs %}
{% code-tabs-item title="app/http/controllers/YourController.py" %}
```python
def show(self, Request):
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

