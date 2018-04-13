# Contracts

## Introduction

Contracts are used when creating drivers to ensure they conform to Masonite requirements. They are a form of interface in other languages where the child class is required to have the minimum number of methods needed for that driver to work. It is a promise to the class that it has the exact methods required.

Contracts are designed to follow the "code to an interface and not an implementation" rule. While this rule is followed, all drivers of the same type are swappable.

Drivers are designed to easily switch the functionality of a specific feature such as switching from file system storage to Amazon S3 storage without changing any code. Because each driver someone creates can technically have whatever methods they want, this creates a technical challenge of having to change any code that uses a driver that doesn't support a specific method. For example a driver that does not have a `store` method while other drivers do will throw errors when a developer switches drivers.

Contracts ensure that all drivers of a similar type such as upload, queue and mail drivers all contain the same methods. While drivers that inherit from a contract can have more methods than required, they should not.

## Getting Started

Contracts are currently used to create drivers and are located in the `masonite.contracts` namespace. Creating a driver and using a contract looks like:

```python
from masonite.contracts.UploadContract import UploadContract

class UploadGoogleDriver(UploadContract):
    pass
```

Now this class will constantly throw exceptions until it overrides all the required methods in the class.

## Contracts

There are several contracts that are required when creating a driver. If you feel like you need to have a new type of driver for a new feature then you should create a contract first and code to a contract instead of an implementation. Below are the types of contracts available. All contracts correspond to their drivers. So an `UploadContract` is required to create an upload driver.

#### BroadcastContract

#### CacheContract

#### MailContract

#### QueueContract

#### UploadContract

