# Masonite 1.6

## Introduction

Masonite 1.6 brings mostly minor changes to the surface layer of Masonite. This release brings a lot of improvements to the engine of Masonite and sets up the framework for the release of 2.0.

## HttpOnly Cookies

Previously, all cookies were set with an HttpOnly flag when setting cookies. This change came after reading several articles about how cookies can be read from Javascript libraries, which is fine, unless those Javascript libraries have been compromised which could lead to a malicious hacker sending your domain name and session cookies to a third party. There is no the ability to turn the HttpOnly flag off when setting cookies by creating cookies like:

```python
request().cookie('key', 'value', http_only=False)
```

## Moved Commands

Because craft is it's own tool essentially and it needs to work across Masonite versions, all commands have been moved into the Masonite repository itself. Now each version of Masonite maintains it's own commands. The new craft version is 2.0

```text
pip install masonite-cli==2.0
```

## Drivers Can Now Change To Other Drivers

Before, you had to use the Manager class associated with a driver to switch a driver. For example:

```python
def show(self, Upload, UploadManager):
    Upload.store(...) # default driver
    UploadManager.driver('s3').store(...) # switched drivers
```

Now you can switch drivers from the driver itself:

```python
def show(self, Upload):
    Upload.store(...) # default driver
    Upload.driver('s3').store(...) # switched drivers
```

## New Absolute Controllers Syntax

This version has been fine tuned for adding packages to Masonite. This version will come along with a new Masonite Billing package. The development of Masonite Billing has discovered some rough spots in package integrations. One of these rough spots were adding controllers that were not in the project. For example, Masonite Billing allows adding a controller that handles incoming Stripe webhooks. Although this was possible before this release, Masonite 1.6 has added a new syntax:

```python
ROUTES = [
    ...
    # Old Syntax:
    Get().route('/url/here', 'Controller@show').module('billing.controllers')
    
    # New Syntax:
    Get().route('/url/here', '/billing.controllers.Controller@show')
    ...
]
```

Notice the new forward slash in the beginning where the string controller goes.

## Added Global Views

Just like the global controllers, some packages may require you to add a view that is located in their package \(like the new exception debug view in 1.6\) so you may now add views in different namespaces:

```python
def show(self):
    return view('/masonite/views/index')
```

This will get a template that is located in the masonite package itself.

## Changed Container Resolving

The container was one of the first features coded in the 1.x release line. For Masonite 1.6 we have revisited how the container resolves objects. Before this release you had to put all annotation classes in the back of the parameter list:

```python
from masonite.request import Request

def show(self, Upload, request: Request):
    pass
```



