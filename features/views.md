Views in Masonite are a great way to return HTML in your controllers. Views have a hierarchy, lots of helpers and logical controls, and a great way to separate out your business logic from your presentation logic.

# Getting Started

By default, all templates for your views are located in the `templates` directory. To create a template, simply create a `.html` file inside this directory which we will reference to later.

```markup
<!-- Template in templates/welcome.html -->
<html>
    <body>
        <h1>Hello, {{ name }}</h1>
    </body>
</html>
```

Then inside your controller file you can reference this template:

```python
from masonite.views import View

class WelcomeController(Controller):

  def show(self, view: View):
    view.render('welcome', {
    	"name": "Joe"
    })
```

The first argument here is the name of your template and the second argument should be a dictionary of variables you reference inside your template.

If you have a template inside another directory you can use dot notation to reference those template names

```python
from masonite.views import View

class WelcomeController(Controller):

  # Template is templates/greetings/welcome.html
  def show(self, view: View):
    view.render('greeting.welcome', {
    	"name": "Joe"
    })
```

# View Sharing

View sharing is when you want a variable to be available in all templates that are rendered. This could be the authenticated user, a copyright year, or anything else you want shared between templates.

View sharing requires you to add a dictionary:

```python
from masonite.facades import View

View.share({
  'copyright': '2021'
})
```

Then inside your template you can do:

```markup
<div>
  Copyright {{ copyright }}
</div>
```

You will typically do this inside your app
[Service Provider](../architecture/service-providers.md) (if you don't have one, you should [create one](../architecture/service-providers.md#creating-a-provider)):

```python
from masonite.facades import View

class AppProvider(Provider):

    def register(self):
        View.share({'copyright': '2021'})
```


# View Composing

Similiar to view sharing, view composing allows you to share templates between specific templates

```python
View.composer('dashboard', {'copyright': '2021'})
```

You may also pass a list of templates:

```python
View.composer(['dashboard', 'dashboard/users'], {'copyright': '2021'})
```

You will typically do this inside your app
[Service Provider](../architecture/service-providers.md) (if you don't have one, you should [create one](../architecture/service-providers.md#creating-a-provider)):

```python
from masonite.facades import View

class AppProvider(Provider):

    def register(self):
        View.composer('dashboard', {'copyright': '2021'})
```

# Helpers

There are quite a few built in helpers in your views. Here is an extensive list of all view helpers:

## Request

You can get the request class:

```markup
<p> Path: {{ request().path }} </p>
```

## Asset

You can get the location of static assets:

You can create a path to an asset by using the `asset` helper:

```markup
...
<img src="{{ asset('s3', 'profile.jpg') }}" alt="profile">
...
```

this will render a URL like this:

```markup
<img src="https://s3.us-east-2.amazonaws.com/bucket/profile.jpg" alt="profile">
```

See your filesystems.py configuration for how how to set the paths up.

## CSRF Field

You can create a CSRF token hidden field to be used with forms:

```markup
<form action="/some/url" method="POST">
    {{ csrf_field }}
    <input ..>
</form>
```

## CSRF Token

You can get only the token that generates. This is useful for JS frontends where you need to pass a CSRF token to the backend for an AJAX call

```markup
<p> Token: {{ csrf_token }} </p>
```

## Current User

You can also get the current authenticated user. This is the same as doing `request.user()`.

```markup
<p> User: {{ auth().email }} </p>
```

This will now submit this form as a PUT request.

## Route

You can get a route by it's name by using this method:

```markup
<form action="{{ route('route.name') }}" method="POST">
    ..
</form>
```

If your route contains variables you need to pass then you can supply a dictionary as the second argument.

```markup
<form action="{{ route('route.name', {'id': 1}) }}" method="POST">
    ..
</form>
```

or a list:

```markup
<form action="{{ route('route.name', [1]) }}" method="POST">
    ..
</form>
```

## Back

This is useful for redirecting back to the previous page. If you supply this helper then the request.back\(\) method will go to this endpoint. It's typically good to use this to go back to a page after a form is submitted with errors:

```markup
<form action="/some/url" method="POST">
    {{ back(request().path) }}
</form>
```

Now when a form is submitted and you want to send the user back then in your controller you just have to do:

```python
def show(self, response: Response):
    # Some failed validation
    return response.back()
```

## Session

You can access the session here:

```markup
<p> Error: {{ session().get('error') }} </p>
```

{% hint style="success" %}
Learn more about session in the [Session](/advanced/sessions.md) documentation.
{% endhint %}

## Config

This allows you to easily fetch configuration values in your templates:

```markup
<h2> App Name: {{ config('application.name') }}</h2>
```


## Cookie

Gets a cookie:

```markup
<h2> Token: {{ cookie('token') }}</h2>
```

## Url

Get the URL to a location:

```markup
<form action="{{ url('/about', full=True) }}" method="POST">

</form>
```

## DD

To help in debugging, you can use the `dd()` helper

```markup
{{ dd(variable) }}
```


# View Filters

Jinja2 allows adding filters to your views. Before we explain how to add filters to all of your templates, let's explain exactly what a view filter is.

Filters can be attached to view variables in order to change and modify them. For example you may have a variable that you want to turn into a slug and have something like:

```python
{{ variable|slug }}
```

In Masonite, this slug filter is simply a function that takes the variable as an argument and would look like a simple function like this:

```python
def slug(variable):
    return variable.replace(' ', '-')
```

That's it! It's important to note that the variable it is filtering is always passed as the first argument and all other parameters are passed in after so we could do something like:

```python
{{ variable|slug('-') }}
```

and then our function would look like:

```python
def slug(variable, replace_with):
    return variable.replace(' ', replace_with)
```

## Adding Filters

Adding filters is typically done inside your app
[Service Provider](../architecture/service-providers.md) (if you don't have one, you should [create one](../architecture/service-providers.md#creating-a-provider)):

```python
from masonite.facades import View


class AppProvider(Provider):

    def register(self):
        View.filter('slug', self.slug)

    @staticmethod
    def slug(item):
        return item.replace(' ', '-')
```


# View Tests

View tests are simply custom boolean expressions that can be used in your templates. We may want to run boolean tests on specific objects to assert that they pass a test. For example we may want to test if a user is an owner of a company like this:

```markup
<div>
    {% if user is a_company_owner %}
        hey boss
    {% else %}
        you are an employee
    {% endif %}
</div>
```

In order to do this we need to add a test on the `View` class. We can once again do this inside the app [Service Provider](../architecture/service-providers.md)

The code is simple and looks something like this:

```python
from masonite.facades import View


def a_company_owner(user):
    # Returns True or False
    return user.owner == 1

class AppProvider(Provider):

    def register(self):
        # template alias
        View.test('a_company_owner', a_company_owner)
```

That's it! Now we can use the `a_company_owner` in our templates just like the first code snippet above!

> Notice that we only supplied the function and we did not instantiate anything. The function or object we supply needs to have 1 parameter which is the object or string we are testing.

# Adding Environments

Environments in views are directories of templates. If you are development packages or building a modular based application, you may have to register different directories for templates. This will allow Masonite to locate your views when referencing them to render. **A good place to do this is inside a service provider's register method.**

There are 2 separate kinds of loaders.

The first loader is a "package" loader. This is used when registering a package. To do this you can simply register it with the package module path. This will work for most directories that are packages.

```python
from masonite.facades import View

View.add('module_name/directory')
```

The other loader is a `FileSystem` loader. This should be used when the directory path is NOT a module but rather just a file system path:

```python
from jinja2.loaders import FileSystemLoader
from masonite.facades import View

View.add(
    os.path.join(
        os.getcwd(), 'package_views'
    )
, loader=FileSystemLoader)
```

# View Syntax

## Jinja2

Below are some examples of the Jinja2 syntax which Masonite uses to build views.

### Line Statements

It's important to note that Jinja2 statements can be rewritten with line statements and line statements are **preferred** in Masonite. In comparison to Jinja2 line statements evaluate the whole line, thus the name line statement.

So Jinja2 syntax looks like this:

```markup
{% if expression %}
    <p>do something</p>
{% endif %}
```

This can be rewritten like this with line statement syntax:

```markup
@if expression
    <p>do something</p>
@endif
```

It's important to note though that these are line statements. Meaning nothing else can be on the line when doing these. For example you CANNOT do this:

```markup
<form action="@if expression: 'something' @endif">

</form>
```

But you could achieve that with the regular formatting:

```markup
<form action="{% if expression %} 'something' {% endif %}">

</form>
```

Whichever syntax you choose is up to you.

> Note that if you are using an `@` symbol that should not be rendered with Masonite then this will throw an error. An example being when you are using `@media` tags in CSS. In this case you will need to wrap this statement inside `{%raw%}` and `{% endraw%}` blocks.

### Variables

You can show variable or string text by using `{{ }}` characters:

```markup
<p>
    {{ variable }}
</p>
<p>
    {{ 'hello world' }}
</p>
```

### If statement

If statements are similar to python but require an endif!

Line Statements:

```markup
@if expression
    <p>do something</p>
@elif expression
    <p>do something else</p>
@else
    <p>above all are false</p>
@endif
```

Using alternative Jinja2 syntax:

```markup
{% if expression %}
    <p>do something</p>
{% elif %}
    <p>do something else</p>
{% else %}
    <p>above all are false</p>
{% endif %}
```

### For Loops

For loop look similar to the regular python syntax.

Line Statements:

```markup
@for item in items
    <p>{{ item }}</p>
@endfor
```

Using alternative Jinja2 syntax:

```markup
{% for item in items %}
    <p>{{ item }}</p>
{% endfor %}
```

### Include statement

An include statement is useful for including other templates.

Line Statements:

```markup
@include 'components/errors.html'

<form action="/">

</form>
```

Using alternative Jinja2 syntax:

```markup
{% include 'components/errors.html' %}

<form action="/">

</form>
```

Any place you have repeating code you can break out and put it into an include template. These templates will have access to all variables in the current template.

### Extends

This is useful for having a child template extend a parent template. There can only be 1 extends per template:

Line Statements:

```markup
@extends 'components/base.html'

@block content
    <p> read below to find out what a block is </p>
@endblock
```

Using alternative Jinja2 syntax:

```markup
{% extends 'components/base.html' %}

{% block content %}
    <p> read below to find out what a block is </p>
{% endblock %}
```

### Blocks

Blocks are sections of code that can be used as placeholders for a parent template. These are only useful when used with the `extends` above. The "base.html" template is the parent template and contains blocks, which are defined in the child template "blocks.html".

Line Statements:

```markup
<!-- components/base.html -->
<html>
    <head>
        @block css
        <!-- block named "css" defined in child template will be inserted here -->
        @endblock
    </head>

<body>
    <div class="container">
        @block content
        <!-- block named "content" defined in child template will be inserted here -->
        @endblock
    </div>

@block js
<!-- block named "js" defined in child template will be inserted here -->
@endblock

</body>
</html>
```

```markup
<!-- components/blocks.html -->
@extends 'components/base.html'

@block css
    <link rel=".." ..>
@endblock

@block content
    <p> This is content </p>
@endblock

@block js
    <script src=".." />
@endblock
```

Using alternative Jinja2 syntax:

```markup
<!-- components/base.html -->
<html>
    <head>
        {% block css %}
        <!-- block named "css" defined in child template will be inserted here -->
        {% endblock %}
    </head>

<body>
    <div class="container">
        {% block content %}
        <!-- block named "content" defined in child template will be inserted here -->
        {% endblock %}
    </div>

{% block js %}
<!-- block named "js" defined in child template will be inserted here -->
{% endblock %}

</body>
</html>
```

```markup
<!-- components/blocks.html -->
{% extends 'components/base.html' %}

{% block css %}
    <link rel=".." ..>
{% endblock %}

{% block content %}
    <p> This is content </p>
{% endblock %}

{% block js %}
    <script src=".." />
{% endblock %}
```

As you see blocks are fundamental and can be defined with Jinja2 and line statements. It allows you to structure your templates and have less repeating code.

The blocks defined in the child template will be passed to the parent template.
