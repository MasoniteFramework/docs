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

## Changed How Controllers Are Created

Previously, controllers were created as they were specified. For example:

```text
$ craft controller DashboardController
```

created a DashboardController. Now the "Controller" part of the controller is appended by default for you. Now we can just specify:

```text
$ craft controller Dashboard
```

to create our DashboardController. You may was to actually just create a controller called Dashboard. We can do this by specifying a flag:

```text
$ craft controller Dashboard -e
```

short for "exact"

## Added Resource Controllers

It's also really good practice to create 1 controller per "action type." For example we might have a `BlogController` and a `PostController`. It's easy to not be sure what action should be in what controllers or what to name your actions. Now you can create a "Resource Controller" which will give you a list of actions such as show, `store`, `create`, `update` etc etc. If what you want to do does not fit with an action you have then you may want to consider making another controller \(such as an `AuthorController`\)

You can now create these Resource Controllers like:

```text
craft controller Dashboard -r
```

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

If we put the annotation in the beginning it would have thrown an error because of how the container resolved.

Now we can put them in any order and the container will grab each one and resolve it.

```python
from masonite.request import Request

def show(self, Upload, request: Request, Broadcast):
    pass
```

This will now work when previously it did not.

## Resolving Instances

The container will now resolve instances of classes as well. It's a common paradigm to "code to an interface and not an implementation." Because of this paradigm, Masonite comes with contracts that act as interfaces but in addition to this, we can also resolve instances of classes. 

For example, all Upload drivers inherit the UploadContract contract so we can simply resolve the UploadContract which will return an Upload Driver:

```python
from masonite.contracts.UploadContract import UploadContract

def show(self, upload: UploadContract):
    upload.store(...)
```

Notice here that we annotated an UploadContract but got back the actual upload driver.

## Removed Some Dependencies

A complaint a few developers pointed out was that Masonite has too many dependencies. Masonite added Pusher, Ably and Boto3 packages by default which added a bit of overhead, especially if developers have no intentions on real time event broadcasting \(which most applications probably won't\). These dependencies have now been removed and will throw an exception if they are used without the required dependencies.

