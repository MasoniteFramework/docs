# View Composers, Sharing, Filters and Tests

## Introduction

Very often you will find yourself adding the same variables to a view again and again. This might look something like

```python
def show(self):
    return view('dashboard', {'request': request()})

def another_method(self):
    return view('dashboard/user', {'request': request()})
```

This can quickly become annoying and it can be much easier if you can just have a variable available in all your templates. For this, we can "share" a variable with all our templates with the `View` class.

The `View` class is loaded into our container under the `ViewClass` alias. It's important to note that the `ViewClass` alias from the container points to the class itself and the `View` from the container points to the `View.render` method. By looking at the `ViewProvider` this will make more sense:

```python
class ViewProvider(ServiceProvider):

    wsgi = False

    def register(self):
        view = View()
        self.app.bind('ViewClass', view)
        self.app.bind('View', view.render)
```

As you can see, we bind the view class itself to `ViewClass` and the render method to the `View` alias.

## View Sharing

We can share variables with all templates by simply specifying them in the `.share()` method like so:

```python
ViewClass.share({'request': request()})
```

The best place to put this is in a new Service Provider. Let's create one now called `ViewComposer`.

```text
$ craft provider ViewComposer
```

This will create a new Service Provider under `app/providers/ViewComposer.py` and should look like this:

```python
class ViewComposer(ServiceProvider):

    def register(self):
        pass

    def boot(self):
        pass
```

We also don't need it to run on every request so we can set `wsgi` to `False`. Doing this will only run this provider when the server first boots up. This will minimize the overhead needed on every request:

```python
class ViewComposer(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self):
        pass
```

Great!

Since we need the request, we can throw it in the `boot` method which has access to everything registered into the service container, including the `Request` class.

```python
class ViewComposer(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self, ViewClass, Request):
        ViewClass.share({'request': Request})
```

Lastly we need to load this into our `PROVIDERS` list inside our `config/application.py` file.

```python
PROVIDERS = [
    # Framework Providers
    ...
    'masonite.providers.ViewProvider',
    'masonite.providers.HelpersProvider',

    # Third Party Providers

    # Application Providers
    'app.providers.UserModelProvider.UserModelProvider',
    'app.providers.MiddlewareProvider.MiddlewareProvider',
    'app.providers.ViewComposer.ViewComposer', # <- New Service Provider
]
```

And we're done! When you next start your server, the `request` variable will be available on all templates.

## View Composing

In addition to sharing these variables with all templates, we can also specify only certain templates. All steps will be exactly the same but instead of the `.share()` method, we can use the `.compose()` method:

```python
def boot(self, ViewClass, Request):
    ViewClass.compose('dashboard', {'request': Request})
```

Now anytime the `dashboard` template is accessed \(the one at `resources/templates/dashboard.html`\) the `request` variable will be available.

We can also specify several templates which will do the same as above but this time with the `resources/templates/dashboard.html` template AND the `resources/templates/dashboard/user.html` template:

```python
def boot(self, ViewClass, Request):
    ViewClass.compose(['dashboard', 'dashboard/user'], {'request': Request})
```

Lastly, we can compose a dictionary for all templates:

```python
def boot(self, ViewClass, Request):
    ViewClass.compose('*', {'request': Request})
```

Note that this has exactly the same behavior as `ViewClass.share()`

## View Filters

Jinja2 allows adding filters to your views. Before we explain how to add filters to all of your templates, let's explain exactly what a view filter is.

### What is a Filter?

Filters can be attached to view variables in order to change and modify them. For example you may have a variable that you want to turn into a slug and have something like:

```python
{{ variable|slug }}
```

In Python, this slug filter is simply a function that takes the variable as an argument and would look like a simple function like this:

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

### Adding Filters

We can add filters simply using the `filter` method on the `ViewClass` class. This will look something like:

```python
class UserModelProvider(ServiceProvider):
    ''' Binds the User model into the Service Container '''

    wsgi = False

    ...

    def boot(self, Request, ViewClass):
        ViewClass.filter('slug', self.slug)

    @staticmethod
    def slug(item):
        return item.replace(' ', '-')
```

{% hint style="info" %}
Make sure that you add filters in a [Service Provider](../architectural-concepts/service-providers.md) that has `wsgi=False` set. This prevents filters from being added on every single request which is not needed.
{% endhint %}

That's it! Adding filters is that easy!

## View Tests

View tests are simply custom boolean expressions that can be used in your templates. We may want to run boolean tests on specific objects to assert that they pass a test. For example we may want to test if a user is an owner of a company like this:

```python
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
...

def a_company_owner(user):
    # Returns True or False
    return user.owner == 1

class SomeProvider:
    wsgi = False

    ...

    def boot(self, view: View):
                  # template alias
        view.test('a_company_owner', a_company_owner)
```

That's it! Now we can use the `a_company_owner` in our templates just like the first code snippet above!

{% hint style="info" %}
Notice that we only supplied the function and we did not instantiate anything. The function or object we supply needs to have 1 parameter which is the object or string we are testing.
{% endhint %}

## Adding Extensions

Jinja2 has the concept of extensions and you can easily add them to your project in a similar way as previous implementations above which is in a [Service Provider](../architectural-concepts/service-providers.md):

```python
class SomeProvider:
    wsgi = False

    ...

    def boot(self, view: View):
        view.add_extension('pypugjs.ext.jinja.PyPugJSExtension')
```

This will add the extension to the view class. 

{% hint style="info" %}
Remember to place this in a service provider where `wsgi=False` as this will prevent the extension being added on every request.
{% endhint %}

