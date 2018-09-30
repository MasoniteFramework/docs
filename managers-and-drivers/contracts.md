# Contracts

## Introduction

Contracts are used when creating drivers to ensure they conform to Masonite requirements. They are a form of interface in other languages where the child class is required to have the minimum number of methods needed for that driver to work. It is a promise to the class that it has the exact methods required.

Contracts are designed to follow the "code to an interface and not an implementation" rule. While this rule is followed, all drivers of the same type are swappable.

Drivers are designed to easily switch the functionality of a specific feature such as switching from file system storage to Amazon S3 storage without changing any code. Because each driver someone creates can technically have whatever methods they want, this creates a technical challenge of having to change any code that uses a driver that doesn't support a specific method. For example a driver that does not have a `store` method while other drivers do will throw errors when a developer switches drivers.

Contracts ensure that all drivers of a similar type such as upload, queue and mail drivers all contain the same methods. While drivers that inherit from a contract can have more methods than required, they should not.

{% hint style="warning" %}
If your driver needs additional methods that can be used that are now inside a contract then your documentation should have that caveat listed in a somewhat obvious manner. This means that by the developer using that new method, they will not be able to switch to other drivers freely without hitting exceptions or having to manually use the methods used by the driver.

Therefore it is advisable to not code additional methods on your drivers and just keep to the methods provided by the base class and contract.
{% endhint %}

## Getting Started

Contracts are currently used to create drivers and are located in the `masonite.contracts` namespace. Creating a driver and using a contract looks like:

```python
from masonite.contracts import UploadContract

class UploadGoogleDriver(UploadContract):
    pass
```

Now this class will constantly throw exceptions until it overrides all the required methods in the class.


## Resolving Contracts

It is useful if you want to "code to an interface and not an implementation." This type of programming paradigm allows your code to be very maintainable because you can simply swap out classes in the container that have the same contract.

For example, Masonite has specific manager contracts depending on the type of driver you are trying to resolve. If we are trying to get the manager for the upload drivers, we can resolve that manager via the corresponding upload manager:

```python
from masonite.contracts import UploadManagerContract

def show(self, upload: UploadManagerContract):
    upload.store(..)
    upload.driver('s3').store(..)
```

Notice this simply returns the specific upload manager used for uploading. Now the upload manager is not a "concrete" implementation but is very swappable. You can load any instance of the the `UploadManagerContract` in the container and Masonite will fetch it for you.

## Contracts

There are several contracts that are required when creating a driver. If you feel like you need to have a new type of driver for a new feature then you should create a contract first and code to a contract instead of an implementation. Below are the types of contracts available. All contracts correspond to their drivers. So an `UploadContract` is required to create an upload driver.

#### BroadcastContract

#### BroadcastManagerContract

#### CacheContract

#### CacheManagerContract

#### MailContract

#### MailManagerContract

#### QueueContract

#### QueueManagerContract

#### UploadContract

#### UploadManagerContract
