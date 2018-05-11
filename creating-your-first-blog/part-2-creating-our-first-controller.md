# Part 2 - Creating Our First Controller

{% hint style="danger" %}
This section is incomplete
{% endhint %}

## Getting Started

All controllers are located in the app/http/controllers directory and Masonite promotes 1 controller per file. This has proven efficient for larger application development because most developers use text editors with advanced search features such as Sublime, VSCode or Atom. Switching between classes in this instance is simple and promotes faster development. It's easy to remember where the controller exactly is because the name of the file is the controller.

You can of course move controllers around wherever you like them but the craft command line tool will default to putting them in separate files.

## Creating Our First Controller

Like most parts of Masonite, you can scaffold a controller with a craft command:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft controller BlogController
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create a controller in `app/http/controllers` that looks like:

{% code-tabs %}
{% code-tabs-item title="app/http/controller/BlogController.py" %}
```python
class BlogController:
    ''' Class Docstring Description '''

    def show(self):
        pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Simple enough, right?

Notice we now have our show method we specified in our route.

## Returning a View

We can return a view from our controller. A view in Masonite are html files. It is what the user will see. We can return a view by using the `view()` function:

```python
def show(self):
    return view('blog')
```

Notice here we didn't import anything. Masonite comes with several helper functions that act like built in Python functions. These helper functions make developing with Masonite really efficient.

{% hint style="success" %}
You can learn more about helper functions in the [Helper Functions](../the-basics/helper-functions.md) documentation
{% endhint %}

## Creating Our View

You'll notice now that we are returning the `blog` view but it does not exist.

All views are in the `resources/templates` directory. We can create a new file called resources/templates/blog.html or we can use another craft command:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft view blog
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will create that file for us.

If we put some text in this file like:

{% code-tabs %}
{% code-tabs-item title="resources/templates/blog.html" %}
```markup
This is a blog
```
{% endcode-tabs-item %}
{% endcode-tabs %}

and then run the server

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft serve
```
{% endcode-tabs-item %}
{% endcode-tabs %}

and open up `localhost:8000/blog`, we will see "This is a blog"

{% hint style="success" %}
In the next part we will start designing our blog application
{% endhint %}

