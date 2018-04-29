# Masonite 1.5 to 1.6

## Introduction

Not much has changed in the actual project structure of Masonite 1.6 so we just need to make some minor changes to our existing 1.5 project

## Requirements.txt

We just have to change our Masonite dependency version in this file:

```text
...
masonite>=1.6,<=1.6.99
```

## Bootstrap/start.py

Masonite 1.6 now wraps the majority of the application in a try and catch block so we can add exception handling such as the new debug view and other exception features down the road such as Sentry and other error logging.

In the middle of the file \(line 45 if you have not made changes to this file\) we can simply wrap the application:

```python
try:
    for provider in container.make('Application').PROVIDERS:
        located_provider = locate(provider)().load_app(container)
        if located_provider.wsgi is True:
            container.resolve(located_provider.boot)
except Exception as e:
    container.make('ExceptionHandler').load_exception(e)
```

This will also display an exception view whenever an exception is it.

## Finished

That's all the changes we have to make for our project.

{% hint style="success" %}
Read [What's New in Masonite 1.6](../whats-new/masonite-1.6.md) for any futher changes you may want or have to make to your code base.
{% endhint %}

