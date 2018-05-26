# About Drivers

## Introduction

Drivers are simply extensions to features that are managed by the Manager Pattern. If we have a `UploadManager` then we might also create a `UploadDiskDriver` and a `UploadS3Driver` which will be able to upload to both the file system \(disk\) and Amazon S3. In the future if we have to upload to Microsoft Azure or Google Cloud Storage then we simply create new drivers like `UploadAzureDriver` and `UploadGoogleStorage` which are very simple to create. Drivers can be as small as a single method or have dozens of methods. The Manager Pattern makes it dead simple to expand the functionality of a Manager and add capabilities to Masonite's features.

## Creating a Driver

Let's go ahead and create a simple driver which is already in the framework called the `UploadDiskDriver`.

If you are creating a driver it can live wherever you like but if you are creating it for Masonite core then it should live inside `masonite/drivers`. For our `UploadDiskDriver` we will create the file: `masonite/drivers/UploadDiskDriver.py`.

We should make a class that looks something like:

```python
class UploadDiskDriver:
    pass
```

{% hint style="warning" %}
**Depending on what type of driver you are making, you may need to inherit from a contract. To ensure this documentation is generalized, we'll leave out contracts for now. Contracts are essentially interfaces that ensure that your driver conforms to all other drivers of the same type. Read more about contracts in the** [**Contracts**](contracts.md) **documentation.**
{% endhint %}

Simple enough, now we can start coding what our API looks like. In the endgame, we want developers to do something like this from their controllers:

```python
def show(self, Upload):
    Upload.store(request().input('file'))
```

So we can go ahead and make a `store` method.

```python
class UploadDiskDriver:

    def store(self, fileitem, location=None):
        pass
```

Ok great. Now here is the important part. Our Manager for this driver \(which is the `UploadManager`\) will resolve the constructor of this driver. This basically means that anything we put in our constructor will be automatically injected into this driver. So for our purposes of this driver, we will need the storage and the application configuration.

```python
class UploadDiskDriver:

    def __init__(self, StorageConfig, Application):
        self.config = StorageConfig
        self.appconfig = Application

    def store(self, fileitem, location=None):
        pass
```

Great. If you're confused about how the dependency injection Service Container works then read the [Service Container](../architectural-concepts/service-container.md) documentation.

Now that we have our configuration we need injected into our class, we can go ahead and build out the `store()` method.:

```python
class UploadDiskDriver:

    def __init__(self, StorageConfig, Application):
        self.config = StorageConfig
        self.appconfig = Application

    def store(self, fileitem, location=None):
        filename = os.path.basename(fileitem.filename)

        if not location:
            location = self.config.DRIVERS['disk']['location']

        location += '/'

        open(location + filename, 'wb').write(fileitem.file.read())

        return location + filename
```

Ok great! You can see that our `store()` method simply takes the file and write the contents of the `fileitem` to the disk.

## Using Our Driver

So now that our driver is created, we can tell our Manager about it. Learn how to create managers under the [About Managers](about-managers.md) documentation. Our manager will know of all drivers that are inside the Service Container. We can create a new service provider which we can use to register classes into our container. Here is an example of what the `UploadProvider` will look like:

```python
from masonite.provider import ServiceProvider
from masonite.managers.UploadManager import UploadManager
from masonite.drivers.UploadDiskDriver import UploadDiskDriver
from config import storage


class UploadProvider(ServiceProvider):

    wsgi = False

    def register(self):
        self.app.bind('StorageConfig', storage)
        self.app.bind('UploadDiskDriver', UploadDiskDriver)
        self.app.bind('UploadManager', UploadManager(self.app))

    def boot(self, UploadManager, StorageConfig):
        self.app.bind('Upload', UploadManager.driver(StorageConfig.DRIVER))
```

Notice how we set our storage configuration in the container, binded our drivers and then binded our Manager. Again, our manager will be able to find all our `UploadXDrivers` that are loaded into the container. So if we set the `DRIVER` inside our configuration file to `google`, our manager will look for a `UploadGoogleDriver` inside our container. Read more about Managers in the [About Managers](about-managers.md) documentation.

That's it! Drivers are extremely simple and most drivers you create will be a simple class with a single method or two.

