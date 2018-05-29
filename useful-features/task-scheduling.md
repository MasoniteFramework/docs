# Task Scheduling

## Introduction

Often your application will require some kind of recurring task that should happen at a specific time of the week, every minute, or every few months. These tasks can be things like:

* Reading XML from a directory and importing it into a remote database
* Checking if a customer is still a subscriber in Stripe and updating your local database
* Cleaning your database of unneeded data every minute or so
* Send an API call to a service in order to fire certain events

Or anything in between. There are lots of use cases for simple tasks to be ran during certain parts of the day or even "offline" hours when your employees are gone. 

{% hint style="warning" %}
This is feature is not a full queue scheduler that can be used with services like Redis or RabbitMQ. This feature is for running simple tasks like the tasks listed above. For queue support, read the [Queues and Jobs](queues-and-jobs.md) documentation.
{% endhint %}

## Getting Started

First we will need to install the  scheduler feature. We can simply pip install it:

```text
$ pip install masonite-scheduler
```

and then add the [Service Provider](../architectural-concepts/service-providers.md) to our `PROVIDERS` list:

```text
PROVIDERS = [
    'masonite.providers.AppProvider',
    ...
    ...
    'scheduler.providers.ScheduleProvider',
]
```

This provider will add several new features to Masonite. The first is that it will add two new commands.

The first command that will be added is the `craft schedule:run` command which will run all the schedules tasks \(which we will create in a bit\). 

The second command is a `craft task` command which will create a new task under the `app/tasks` directory.

## Creating a Task

Now that we added the Service Provider, we can start creating tasks. Let's create a super basic task that prints "Hi". First let's create the task itself:

```text
$ craft task SayHi
```

This will create a file under app/tasks/SayHi.py

```python
from scheduler.Task import Task


class SayHi(Task):

    def __init__(self):
        pass
    
    def handle(self):
        pass

```

This will be the simple boiler plate for our tasks.

All tasks should inherit the `scheduler.task.Task` class. This adds some needed methods to the task itself but also allows a way to fetch all the tasks from the [container by collecting them](../architectural-concepts/service-container.md#collecting).

{% hint style="success" %}
Make sure you read about collecting objects from the container by reading the [Collecting](../architectural-concepts/service-container.md#collecting) section of the [Service Container](../architectural-concepts/service-container.md) documentation.
{% endhint %}

## Container Autoloading

In order for tasks to be discovered they need to be inside the container. Once inside the container, we collect them, see if they need to run and then decide to execute them or not.

There are two ways to get classes into the container. The first is to [bind them into the container](../architectural-concepts/service-container.md#bind) manually by creating a [Service Provider](../architectural-concepts/service-providers.md). You can read the documentation on creating [Service Providers](../architectural-concepts/service-providers.md) if you don't know how to do that.

The other way is to [Autoload](../advanced/autoloading.md) them. Starting with Masonite 2.0, You can autoload entire directories which will find classes in that directory and load them into the container. This can be done by adding the directory your tasks are located in to the AUTOLOAD config variable inside config/application.py:

```text
AUTOLOAD = [
    'app',
    'app/tasks'
]
```

This will find all the tasks in the app/tasks directory and load them into the container for you with the key binding being the name of the class.

## Making The Task

Now that our task is able to be added to the container automatically, let's start building the class. 

### Constructors

Firstly, the constructor of all tasks are resolved by the container. You can fetch anything from the container that doesn't need the WSGI server to be running \(which is pretty much everything\). So we can fetch things like the Upload, Mail, Broadcast and Request objects. This will look something like:

```python
from scheduler.Task import Task
from masonite.request import Request


class SayHi(Task):

    def __init__(self, request: Request):
        self.request = request
    
    def handle(self):
        pass
```

You can either use the annotations here or the usual resolving by the key name:

```python
from scheduler.Task import Task


class SayHi(Task):

    def __init__(self, Request):
        self.request = Request
    
    def handle(self):
        pass
```

### Handle Method

The handle method is where the logic of the task should live. This is where you should put the logic of the task that should be running recurrently.

We can do something like fire an API call here:

```python
from scheduler.Task import Task
import requests


class SayHi(Task):

    def __init__(self):
        pass
    
    def handle(self):
        requests.post('http://url.com/api/store')
```

### When To Run

The awesomeness of recurring tasks is telling the task when it should run. There are a few options we can go over here:

A complete task could look something like:

```python
from scheduler.Task import Task
import requests


class SayHi(Task):

    run_every = '3 days'
    run_at = '17:00'

    def __init__(self):
        pass
    
    def handle(self):
        requests.post('http://url.com/api/store')
```

This task will fire that API call every 3 days at 5pm.

All possible options are `False` by default. The options here are:

| **Attribute** | **Options** | **Example** |
| --- | --- | --- | --- | --- | --- |
| run\_every | Either a singular or plural version of the accepted time units: `minutes`, `hours`, `days`, `months` | run\_every = '1 day' |
| run\_at | The time in military time \("17:00" for 5pm\) | run\_at = '17:00' |
| run\_every\_hour | Boolean on whether to run every hour or not: `True`, `False` | run\_every\_hour = True |
| run\_every\_minute | Boolean on whether to run every minute or not: `True`, `False` | run\_every\_minute = True |
| twice\_daily | A tuple on the hours of the day the task should run. Also in military time. `(1, 13)` will run at 1am and 1pm. | twice\_daily = \(1, 13\) |

{% hint style="info" %}
If the time on the task is `days` or `months` then you can also specify a `run_at` attribute which will set the time of day it should should. By default, all tasks will run at midnight if `days` is set and midnight on the first of the month when `months` is set. We can specify which time of day using the `run_at` attribute along side the `run_every` attribute. This option will be ignored if `run_every` is `minutes` or `hours`. 
{% endhint %}

## Caveats

{% hint style="warning" %}
This feature is designed to run without having to spin up a seperate server command and therefore has some caveats so be sure to read this section to get a full understanding on how this feature works.
{% endhint %}

### When it Runs

Since the scheduler doesn't actually know when the server starts, it doesn't know from what day to start the count down. In order to get around this, Masonite calculates the current day using a modulus operator to see if the modulus of the tasks time and the current time are 0.

For example, if the task above is to be ran \(every 3 days\) in May then the task will be ran at midnight on May 3rd, May 6th, May 9th, May 12th etc etc. So it's important to note that if the task is created on May 11th and should be ran every 3 days, then it will run the next day on the 12th and then 3 days after that.

## Running The Tasks

After we add the directory to the `AUTOLOAD` list, we can run the `schedule:run` command which will find the command and execute it.

```text
$ craft schedule:run
```

Masonite will fetch all tasks from the container by finding all subclasses of scheduler.tasks.Task, check if they should run and then either execute it or not execute it.  
  
Even though we ran the task, we should not see any output. Let's change the task a bit by printing "Hi" and setting it to run every minute:

```python
from scheduler.Task import Task


class SayHi(Task):

    run_every = '1 minute'

    def __init__(self):
        pass
    
    def handle(self):
        print('Hi!')
```

Now let's run the command again:

```text
$ craft schedule:run
```

We should now see "Hi!" output to the terminal window.

## Cron Jobs

{% hint style="warning" %}
Mac Only
{% endhint %}

Although the command above is useful, it is not very practical in a production setup. In production, we should setup a cron job to run that command every minute so Masonite can decide on what jobs need to be ran.

We'll show you an example cron job and then we will walk through how to build it.

```text
PATH=/Users/Masonite/Programming/project_name/venv/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Frameworks/Python.framework/Versions/3.6/bin
* * * * * cd /Users/Masonite/Programming/project_name && source venv/bin/activate && craft schedule:run
```

### Getting The Path

When a cron job runs, it will typically run commands with a /bin/sh command instead of the usual /bin/bash. Because of this, craft may not be found on the machine so we need to tell the cron job the PATH that should be loaded in. We can simply find the PATH by going to our project directory and running:

```text
$ env
```

Which will show an output of something like:

```text
...
__CF_USER_TEXT_ENCODING=0x1F5:0x0:0x0
PATH=/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Frameworks/Python.framework/Versions/3.6/bin
PWD=/Users/Masonite/Programming/masonite
...
```

{% hint style="info" %}
If you are using a virtual environment for development purposes then you need to run the `env` command inside your virtual environment.
{% endhint %}

We can then copy the PATH and put it in the cron job.

To enter into cron, just run:

```text
$ env EDITOR=nano crontab -e
```

and paste the `PATH` we just copied. Once we do that our cron should look like:

```text
PATH=/Users/Masonite/Programming/masonitetesting/venv/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Frameworks/Python.framework/Versions/3.6/bin
```

Exit out of nano. Now we just need to setup the actual cron job:

### Setting The Cron Task

Now we just need to setup the cron task itself. This could be as easy as copying it and pasting it into the nano editor again. You may need to change a few things though.

The first `* * * * *` part is a must and bascially means "run this every minute by default". The next part is the location of your application which is dependant on where you installed your Masonite application.

The next part is dependant on your setup. If you have a virtual environment then you need to activate it by appending `&& source venv/bin/activate` to the cron task. If you are not running in a virtual environment then you can leave that part out.

Lastly we need to run the schedule command so we can append `&& craft schedule:run`

Great! Now we have a cron job that will run the craft command every minute. Masonite will decide which classes need to be executed.



