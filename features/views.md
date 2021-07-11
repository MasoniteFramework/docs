Views in Masonite are a great way to return HTML in your controllers. Views have a hierarchy, lots of helpers and logical controls, and a great way to separate out your business logic from your presentation logic.

# Getting Started

By default, all templates for your views are located in the `templates` directory. To create a template, simply create a `.html` file inside this directory which we will reference to later.

```html
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

```html
<div>
  Copyright {{ copyright }}
</div>
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

We can add filters simply using the `filter` method on the `ViewClass` class. This will look something like:

```python
from masonite.view import View

class UserModelProvider(ServiceProvider):
    """Binds the User model into the Service Container."""

    wsgi = False

    ...

    def boot(self, view: View):
        view.filter('slug', self.slug)

    @staticmethod
    def slug(item):
        return item.replace(' ', '-')
```

> Make sure that you add filters in a [Service Provider](../architectural-concepts/service-providers.md) that has `wsgi=False` set. This prevents filters from being added on every single request which is not needed.

# View Tests

View tests are simply custom boolean expressions that can be used in your templates. We may want to run boolean tests on specific objects to assert that they pass a test. For example we may want to test if a user is an owner of a company like this:

```html
<div>
    {% if user is a_company_owner %}
        hey boss
    {% else %}
        you are an employee
    {% endif %}
</div>
```

In order to do this we need to add a test on the `View` class. We can do this in a Service Provider. The Service Provider you choose should preferably have a `wsgi=False` attribute so the test isn't added on every single request which could potentially slow down the application.

The code is simple and looks something like this:

```python
from masonite.view import View
# ...

def a_company_owner(user):
    # Returns True or False
    return user.owner == 1

class SomeProvider(Provider):
    # ...
    def register(self, view: View):
                  # template alias
        view.test('a_company_owner', a_company_owner)
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