# Views

# Introduction

Views contain all the HTML that you’re application will use to render to the user. Unlike Django, views in Masonite are your HTML templates. All views are located inside `resources/templates`

All views are rendered with Jinja2 so can use all the Jinja2 code you are used to. An example view looks like:

```html
<!-- View stored in resources/templates/helloworld.html -->

<html>
  <body>
    <h1> Hello {{ 'world' }}</h1>
  </body>
</html>
```

## Creating Views

Since all views are located in `resources/templates`, we can use simply create all of our views manually here or use our `craft` command tool. To make a view just run:

```
$ craft view hello
```

This will create a template under `resources/templates/hello.html`.

## Calling Views

### Helper Function

There are several ways we can call views in our controllers. The first recommended way is using the `view()` function. Masonite ships with a `HelpersProvider` Service Provider. This provider will add several new built in functions to your project. These helper functions can be used as shorthand for several commonly used classes such as the `View` and `Request` class. See the "Helper Functions" documentation for more information.

One of the helper functions is the `view()` function which is accessible like any other built in Python function.

We can call views in our controllers like so:

```python
def show(self):
    return view('dashboard')
```

This will return the view located at `resources/templates/dashboard.html`. We can also specify a deeper folder structure like so:

```python
def show(self):
    return view('profiles/dashboard')
```

This will look for the view at `resources/templates/profiles/dashboard.html`

### From The Container

The `View` class is loaded into the container so we can retrieve it in our controller methods like so:

```python
def show(self, View):
    return View('dashboard')
```

This is exactly the same as using the helper function above. So if you choose to code more explicitly, the option is there for you.

## Passing Data to Views

A lot of the time we’ll need to pass in data to our views. This data is passed in with a dictionary that contains a key which is the variable that the view will with the corresponding value. We can pass data to the function like so:

```python
def show(self, Request):
    return view('dashboard', {'id': Request.param('id')})
```

**Remember that by passing in parameters like **`Request`** to the controller method, we can retrieve objects from the IOC container. Read more about the IOC container in the "Service Container" documentation.**

This will send a variable named `id` to the view which can then be rendered like:

```html
<!-- View stored in resources/templates/dashboard -->

<html>
  <body>
    <h1> {{ id }} </h1>
  </body>
</html>
```



