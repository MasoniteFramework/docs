# Task Scheduling

## Introduction

Often your application will require some kind of recurring task that should happen at a specific time of the week, every minute, or every few months. These tasks can be things like:

* Reading XML from a directory and importing it into a remote database
* Checking if a customer is still a subscriber in Stripe and updating your local database
* Cleaning your database of unneeded data every minute or so
* Send an API call to a service in order to fire certain events

Or anything in between. There are lots of use cases for simple tasks to be ran during certain parts of the day or even "offline" hours when your employees are gone.

## Getting Started

First we will need to install the scheduler feature. We can simply pip install it:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ pip install masonite-scheduler
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Masonite will fetch all tasks from the container by finding all subclasses of `scheduler.tasks.Task`, check if they should run and then either execute it or not execute it.

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

### Running a Specific Task

You may also run a specific task by running the schedule:run command with a --task flag. The flag value is the container binding \(usually the task class name\):

```text
 craft schedule:run --task SayHi
```

Or you can give your task a name explicitly:

```python
from scheduler.Task import Task


class SayHi(Task):

    run_every = '1 minute'
    name = 'hey'

    def __init__(self):
        pass

    def handle(self):
        print('Hi!')
```

and then run the command by name

```text
 craft schedule:run --task hey
```

## Cron Jobs

{% hint style="warning" %}
Setting up Cron Jobs are for UNIX based machines like Mac and Linux only. Windows has a similar schedule called Task Scheduler which is similar but will require different instructions in setting it up.
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

