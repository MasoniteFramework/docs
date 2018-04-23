# Template Caching

## Introduction

Sometimes your templates will not change that often and may have a lot of logic that takes times to run such as several loops or even a very large order of magnitude. If this page is being hit several times per day or even several times per second, you can use template caching in order to put less strain on your server.

This is a powerful feature that will reduce the load of your server for those resource hungry and complex templates.

## Getting Started

This feature is introduced in Masonite 1.4 and above. You can check your Masonite version by running `pip show masonite` which will give you the details of the masonite package you have installed.

You can use whatever cache you like for caching templates. Only the `disk` driver is supported out of the box but you can create any drivers you like. Read the [About Drivers](../managers-and-drivers/about-drivers.md) documentation on how to create drivers. If you do create a driver, consider making it available on PyPi so others can install it into their projects. If you'd like to contribute to the project and add to the drivers that come freshly installed with Masonite then please visit Masonite's GitHub repository and open an issue for discussing.

All caching configuration is inside `config/cache.py` which contains two settings, `DRIVER` and `DRIVERS`.

We can set the driver we want to use by specifying like so:

```python
DRIVER = 'disk'
```

Like every other configuration, you'll need to specify the options inside the `DRIVERS` dictionary:

```python
DRIVERS = {
    'disk': {
        'location': 'bootstrap/cache'
    }
}
```

All templates cached will be inside that folder. You may wish to specify another directory for your templates such as:

```python
DRIVERS = {
    'disk': {
        'location': 'bootstrap/cache/templates'
    }
}
```

## Caching Templates

In order to cache templates for any given amount of time, you can attach the `.cache_for()` method onto your view. This looks like:

```python
def show(self):
    return view('dashboard/user').cache_for(5, 'seconds')
```

This will cache the template for 5 seconds. After 5 seconds, the next hit on that page will show the non cached template and then recache for another 5 seconds.

What has always been annoying in many libraries and frameworks is the distinguishhment between plural and singular such as `second` and `seconds`. So if we only want to do 1 minute then that would look like:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'minute')
```

### Options

There are several cache lengths we can use. Below is all the options for caching:

We can cache for several seconds:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'second')

def show(self):
    return view('dashboard/user').cache_for(10, 'seconds')
```

or several minutes:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'minute')

def show(self):
    return view('dashboard/user').cache_for(10, 'minutes')
```

or several hours:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'hour')

def show(self):
    return view('dashboard/user').cache_for(10, 'hours')
```

or several days:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'days')

def show(self):
    return view('dashboard/user').cache_for(10, 'days')
```

or several months:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'month')

def show(self):
    return view('dashboard/user').cache_for(10, 'months')
```

or even several years:

```python
def show(self):
    return view('dashboard/user').cache_for(1, 'year')

def show(self):
    return view('dashboard/user').cache_for(10, 'years')
```

