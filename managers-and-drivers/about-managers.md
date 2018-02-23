# About Managers

# Introduction

Masonite uses an extremely powerful pattern commonly known as the Manager Pattern; also known as the Builder Pattern. Because Masonite uses classes with the `XManager` namespace, we will call it the Manager Pattern throughout this documentation.

Think of the Manager Pattern as attaching a Manager to a specific feature. This Manager is responsible for instantiating `FeatureXDriver` classes. For example, we attach a `UploadManager` to the upload feature. Now the `UploadFeature` will instantiate `UploadXDriver` classes.

For an actual example inside Masonite, there are currently two classes for the Upload feature: `UploadDiskDriver` and `UploadS3Driver`. Whenever we set the `DRIVER` in our `config/storage.py` file to `s3`, the `UploadManager` will use the `UploadS3Driver` to store our files.

This is extremely useful for extending functionality of the managers. If we need to upload to Google, we can just make a `UploadGoogleDriver` and put it inside the container. If we set our configuration `DRIVER` to `google`, our `UploadManager` will now use that class to store files.

## Creating a Manager

Masonite obviously comes with several managers such as the `UploadManager` and the `MailManager`. Let's walk through how to create a new manager called the `TaskManager`.

Managers can live wherever they want but if you are developing a manager for the Masonite core package, they will be placed inside `masonite/managers`.

Let's create a new file: `masonite/managers/TaskManager.py`.

Great! Now all managers should inherit from the `masonite.managers.Manager` class. Our `TaskManager` should look something like:

```python
from masonite.managers.Manager import Manager

class TaskManager(Manager):
    pass
```

Awesome! Inheriting from the Manager class will give our manager almost all the methods it needs. The only thing we need now is to tell this manager how to create drivers. So do to this we need a `create_driver()` method.

```python
from masonite.managers.Manager import Manager

class TaskManager(Manager):
    
    def create_driver(self):
        pass
```

Perfect. Now the logic of this create driver should be pretty straight forward and should be almost identical to all other managers because we are simply instantiating drivers with different namespaces:

```python
from masonite.managers.Manager import Manager
from masonite.exceptions import DriverNotFound

class TaskManager(Manager):

    def create_driver(self, driver=None):
        if not driver:
            driver = self.container.make('TaskConfig').DRIVER.capitalize()
        else:
            driver = driver.capitalize()

        try:
            self.manage_driver = self.container.make(
                'Task{0}Driver'.format(driver))
        except KeyError:
            raise DriverNotFound(
                'Could not find the Task{0}Driver from the service container. Are you missing a service provider?'.format(driver))
```

Ok so that was a bit of code. Although it's pretty straight forward, let's explain what's happening here.

So if the driver doesn't exist, we need to create a new driver depending on what's inside the configuration file loaded inside the container. This makes sense because we need to create a driver based off of something. If the developer didn't supply us with one then we need to grab it from the configuration file. Since we are using `TaskConfig` here we will eventually have to load it into the container. In a Service Provider, this might look something like:

```python
from config import task

container.bind('TaskConfig', task)
```

So we will need to load that into the Service Container in order for this manager to work. We will create a Service Provider later on but for now just know that that's where we get that configuration from. We also capitalize the string to ensure consistency across drivers.

Next we simply call the `TaskXDriver` from the container and set that as the driver we want to manage which is done by setting the `manage_driver` class attribute.

If we cannot find the driver in the container then we have a `KeyError` and we call the `DriverNotFound` exception which we imported above. This is only called when the `DRIVER` we set in our configuration does not have a corresponding driver inside the container. For example if the developer sets `DRIVER='todo'` and we have not set the `TaskTodoDriver`

## Using Our Manager

We can use our manager simply by loading it into the container. We can do this by creating a Service Provider. Learn more about how to create a Service Provider in the [Service Providers](/service-container/service-providers.md) documentation. Let's show what a basic Service Provider might look like:

```python
from masonite.provider import ServiceProvider
from masonite.drivers.TaskTodoDriver import TaskTodoDriver
from masonite.managers.TaskManager import TaskManager
from config import task

class TaskProvider(ServiceProvider):

    wsgi = False

    def register(self):
        self.app.bind('TaskConfig', task)
        self.app.bind('TaskTodoDriver', TaskTodoDriver)
        self.app.bind('TaskManager', TaskManager(self.app))

    def boot(self, TaskManager, TaskConfig):
        self.app.bind('Task', TaskManager.driver(TaskConfig.DRIVER))
```

Great! We can put this Service Provider in our `app/application.py` file inside the `PROVIDERS` list. Once that is inside our providers list we can now use this new `Task` alias in our controllers like so:

```python
def show(self, Task):
    Task.method_here()
```

Read about how to create drivers for your Manager class under the [About Drivers](/managers-and-drivers/about-drivers.md) documentation.





