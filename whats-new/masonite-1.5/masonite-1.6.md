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

```text
def show(self, Upload, UploadManager):
    Upload.store(...) # default driver
    UploadManager.driver('s3').store(...) # switched drivers
```

Now you can switch drivers from the driver itself:

```text
def show(self, Upload):
    Upload.store(...) # default driver
    Upload.driver('s3').store(...) # switched drivers
```

