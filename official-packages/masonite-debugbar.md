# Masonite Debugbar

Masonite Debugbar is a really helpful way to see the stats of your application while you are developing. You can use this information to help debug errors you are getting or even optimize your models and queries for speed and memory issues.

Masonite Debugbar also supports AJAX calls which you will be able to see directly in a dropdown on your toolbar.

![](<../.gitbook/assets/Screen Shot 2022-01-23 at 2.26.56 PM.png>)

## Install

Setting up Masonite Debugbar is simple.

First, install the package:

```
$ pip install masonite-debugbar
```


Put the provider at the end of your provider list:

```python
from debugbar.providers import DebugProvider
PROVIDERS = [
    #.. 
    DebugProvider
]
```

Then publish the package:

```
$ python craft package:publish debugbar
```

{% hint style="warning" %}
Finally ensure [debug mode](/the-basics/environments#debug-mode) is enabled else the debugbar will not be displayed !
{% endhint %}

Now when you go to a page in your application, you will see a debug bar at the bottom of the page.

## Config

The configuration file is created on collectors

```python
OPTIONS = {
    "models": True,
    "queries": True,
    "request": True,
    "measures": True,
    "messages": True,
    "environment": True,
}
```

Not all collectors may be equally important to you so you can set anyone of these to either `True` or `False` in order to enable or disable them in the debugbar.

## Collectors

Masonite Debugbar collects data to show in your debugbar as a tab. Each collector is 1 tab in the debugbar.

Below is a detailed explanation of each collector

### Model Collector

The model collector shows how many models are hydrated on that request. Whenever you make a query call, a model instance has to be created to store that rows data. This could be a costly operation depending on how many rows are in the table you are calling.

### Queries Collector

The queries collector shows all the queries made during that request as well as the time it took to perform that query. Use this to see where bottle necks are in your application. Slow queries lead to slow load times for your users.

### Request Collector

The request collector shows you information related to the request such as inputs, parameters and headers.

### Message Collector

The message collector contains messages you can add in your application. Instead of adding a bunch of print statements you can add a message:

```python
from debugbar.facades import Debugbar

Debugbar.get_collector('messages').add_message("a debug message")
```

You could also add tags which will create a colored tag in the content tab:

```python
Debugbar.get_collector('messages').add_message("a debug message", tags={"color": "green", "message": "tag name"})
```

### Environment Collector

This collector adds all of your environment variables to your debugbar as well as the Python and Masonite versions.

### Measures Collector

The measures collector you can use to measure 2 points in your application. By default there is the time it takes for your application to complete the whole request. You can start and stop any measures you want:

```python
from debugbar.facades import Debugbar

Debugbar.start_measure("loop check")
# .. Long running code
Debugbar.stop_measure("loop check")
```

You will now see the time it takes to run this code in the measures tab

## Adding Your Own Collectors

If you find a need to create your own collector, maybe to log information related to exceptions or something similar, you can create your own collector simply:

### Creating the Collector

Collectors are simple instances like this:

```python
class YourCollector:
    def __init__(self, name="Your Collector", description="Description"):
        self.messages = []
        self.name = name
        self.description = description

    def restart(self):
        self.messages = []
        return self
```

The restart method is required to restart your collector have each request so the information doesn't get persisted bewteen requests. If this is not required for your collector then you can simply return `self`.

### Collector Methods

The next part you'll need is a way to add data to your collector. This can be done in any method you want your developers to use. These are your external API's and how you want developers interacting with your collector. You can name these methods whatever you want and can be as complex as you need them to be.

```python
class YourCollector:
    def __init__(self, name="Your Collector", description="Description"):
        self.messages = []
        self.name = name
        self.description = description

    def restart(self):
        self.messages = []
        return self

    def add(self, key, value):
        self.messages.append({key: value})
```

This method could be as simple or as complex as you need. Some of Masonite Debugbar's collectors use special classes to keep all the information.

### Collector Rendering

Next you need a `collect` and an `html` method to finalize the collector. First the collect method should just return a dictionary like this:

```python
from jinja2 import Template


class YourCollector:
    def __init__(self, name="Your Collector", description="Description"):
        self.messages = []
        self.name = name
        self.description = description

    def restart(self):
        self.messages = []
        return self

    def add(self, key, value):
        self.messages.append({key: value})
    
    def collect(self):
        collection = []
        for message in self.messages:
            for key, value in message.items():
                collection.append(
                    {
                        "key": key,
                        "value": value
                    }
                )
        # # render data to html
        template = Template(self.html())
        return {
            "description": self.description,
            "count": len(collection),
            "data": collection,
            "html": template.render({"data": collection}),
        }
    
    def html(self):
        return """
        <div>
            <div class="flex justify-between p-4">
                <p>{{ object.key }}: {{ object.value }}</p>
            </div>
        </div>
        """
```

`data` here in the html method loop is the collection we built. You can use this to style your tab and content.

In the collect method return dictionary, the `description` is used to show a quick description at the top of the tabs content. The `count` here is the number badge shown in the actual tab itself.

### Registering the collector

Lastly we just need to register the collector to the debugbar for it to show up. You can do this in your own provider. If you are building this in your own application you can make your own provider. If you are building a collector as part of a package you can have your developers install it in their projects.

```python
class YourProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        debugger = self.application.make('debugger')
        debugger.add_collector(YourCollector())

    def boot(self):
        pass
```

And then finally register the provider with your provider config and your collector will now show up in the debug toolbar with the rest of the collectors.
