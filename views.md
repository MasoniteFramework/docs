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

We can call views in our controllers using the `view()` function. The `view()` function is imported for us whenever we run the `craft controller` command. If you did not run the `craft controller` command then we can import this function using `from masonite.view import view`

We can call views in our controllers like so:

```python
def show(self):
    return view('dashboard')
```

This will return the view located in `resources/templates/dashboard.html`. We can also specify a deeper folder structure like so:

```python
def show(self):
    return view('profiles/dashboard')
```

This will look for the view inside `resources/templates/profiles/dashboard`

## Passing Data to Views

A lot of the time we’ll need to pass in data to our views. This data is passed in with a dictionary that contains a key which is the variable that the view will use as well as the value. We can pass data to the function like:

```python
def show(self, Request):
    return view('dashboard', {'id': Request.param('id')})
```

Remember that by passing in values to the controller method, we can retrieve objects from the IOC container. Read more about the IOC container in the "Service Provider" documentation.

This will send a variable named `id` to the view which can then be rendered like:

```html
<!-- View stored in resources/templates/dashboard -->

<html>
  <body>
    <h1> {{ id }} </h1>
  </body>
</html>
```



