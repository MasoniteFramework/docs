# About Managers

## Introduction

Masonite uses an extremely powerful pattern commonly known as the Manager Pattern \(also known as the Builder Pattern\). Because Masonite uses classes with the `XManager` namespace, we will call it the Manager Pattern throughout this documentation.

Think of the Manager Pattern as attaching a Manager to a specific feature and responsible for managing a specific set of drivers. These managers are responsible for instantiating `FeatureXDriver` classes. For example, we attach a `UploadManager` to the upload feature. Now the `UploadFeature` will instantiate `UploadXDriver` classes.

For an actual example inside Masonite, there are currently two classes for the Upload feature: `UploadDiskDriver` and `UploadS3Driver`. Whenever we set the `DRIVER` in our `config/storage.py` file to `s3`, the `UploadManager` will use the `UploadS3Driver` to store our files.

This is extremely useful for extending functionality of the managers. If we need to upload to Google, we can just make a `UploadGoogleDriver` and put it inside the container. If we set our configuration `DRIVER` to `google`, our `UploadManager` will now use that class to store files.

## Creating a Manager

Masonite obviously comes with several managers such as the `UploadManager` and the `MailManager`. Let's walk through how to create a new manager called the `TaskManager`.

Managers can live wherever they want but if you are developing a manager for the Masonite core package, they will be placed inside `masonite/managers`.

Let's create a new file: `masonite/managers/TaskManager.py`.

Great! Now all managers should inherit from the `masonite.managers.Manager` class. Our `TaskManager` should look something like:

```python
from masonite.managers import Manager

class TaskManager(Manager):
    pass
```

Awesome! Inheriting from the Manager class will give our manager almost all the methods it needs. The only thing we need now is to tell this manager how to create drivers. So to do this all we need are two attributes:

```python
from masonite.managers import Manager

class TaskManager(Manager):

    config = 'TaskConfig'
    driver_prefix = 'Task'
```

Perfect. Managers are both extremely powerful and easy to create. That's it. That's our entire manager. The config attribute is the configuration file you want which via the key in the container and the `driver_prefix` is the drivers you want to manage. In this case it is the `Task{X}Driver`. This manager will manage all drivers in the container that conform to the namespaces of `Task{0}Driver` like `TaskTodoDriver` and `TaskNoteDriver`.

Notice that the config is `TaskConfig` and not `task`. This attribute is the binding name and not the config name. We can bind the `task` config into the container like so:

```python
from config import task

container.bind('TaskConfig', task)
```

Which will be required to use our new task manager since it relies on the task configuration. You can do this inside the Service Provider that will ship with this manager. We will create a Service Provider later on but for now just know that that's where that configuration comes from.

## Using Our Manager

We can use our manager simply by loading it into the container. We can do this by creating a Service Provider. Learn more about how to create a Service Provider in the [Service Providers](../architectural-concepts/service-providers.md) documentation. Let's show what a basic Service Provider might look like:

```python
from masonite.provider import ServiceProvider
from masonite.drivers import TaskTodoDriver
from masonite.managers import TaskManager
from config import task

class TaskProvider(ServiceProvider):

    wsgi = False

    def register(self):
        self.app.bind('TaskConfig', task)
        self.app.bind('TaskTodoDriver', TaskTodoDriver)
        self.app.bind('TaskManager', TaskManager(self.app))

    def boot(self, manager: TaskManager):
        self.app.bind('Task', manager.driver(task.DRIVER))
```

Great! We can put this Service Provider in our `app/application.py` file inside the `PROVIDERS` list. Once that is inside our providers list we can now use our new manager:

```python
from masonite.managers import TaskManager

def show(self, manager: TaskManager):
    manager.driver('todo').method_here()
```

## Container Swapping

Although the above code works fine it might be more useful to add a container swap in our Service Provider so we can resolve an arbitrary shorthand class which will return our correct driver. We can do this in our boot method:

```python
class Task:
    pass

..
    def boot(self, manager: TaskManager):
        self.app.bind('Task', manager.driver(task.DRIVER))
        self.app.swap(Task, manager.driver(task.DRIVER))
```

Now we can resolve this `Task` class which will return the correct driver since we specified a driver swap:

```python
from somewhere import Task

def show(self, task: Task):
    task.method_here()
```

{% hint style="success" %}
Read about how to create drivers for your Manager class under the [About Drivers](about-drivers.md) documentation.
{% endhint %}

