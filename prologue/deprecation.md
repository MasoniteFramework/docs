# Deprecation

## Introduction

This page contains all deprecated code that may be removed in future versions of Masonite. These helpers are available to use for the remainder of the 2.1 lifetime but will be removed in either the next version \(2.2\) or the version after.

## About Deprecation

Some code inside Masonite becomes obsolete as new features become available. In order to make maintaining the codebase easier and clutter free, Masonite will start emitting deprecation warnings during minor releases of the current release.

For example if you upgrade from 2.1.20 to 2.1.21, you may start seeing deprecation warnings inside your console. You can safely ignore these warnings and your code will still work fine but may not work when upgrading to the next major version \(for example from 2.1 to 2.2\).

## What to do

When you see these deprecation warnings, you will see a link as well that comes to this page. You should scroll down this page for examples on how to get your code up to date and not in a deprecated state. You should do this as soon as it becomes convenient for you to do so.

## Deprecated Code

Below is a list of all code that is being deprecated in 2.1 and may be removed in 2.2 and later releases.

### Helper Functions

Helper functions have now been deprecated. A feature release came out that allows passing routes and controllers to the constructors of route classes. This effectively made route helpers obsolete and there is no reason to use them anymore. You may now use normal route classes.

If you have any code that looks like this:

```python
from masonite.helpers.routes import get, post

ROUTES = [
    get('/some/url', 'Controller@show'),
    post('/another/url', 'Controller@store'),
]
```

You can now simply import the route class name and use alias at the top. Doing so will allow you to not have to change any code in your route list:

```python
from masonite.helpers.routes import Get as get, Post as post

ROUTES = [
    get(..),
    post(..),
]
```

