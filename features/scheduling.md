Masonite ships with an incredibly simple way to run recurring tasks. Tasks could be as simple as cleaning records on a database table every minute, or syncing records between databases, or sending invoices out at the end of the month. They are those automated recurring tasks you need to run on a schedule, like every minute, every hour, every day, every month, or anywhere in between.

# Creating Tasks

Tasks are what you use to register to the Masonite scheduler do it knows which tasks to run and how often.

To create a task, simply run the command:

```terminal
$ python craft task SendInvoices
```

This will create a task that looks like this:

```python
from masonite.scheduling import Task

class SendInvoices(Task):
    def handle(self):
        pass
```

You can change the task to do whatever you need it to do:

```python
from masonite.scheduling import Task

class SendInvoices(Task):
    def handle(self):
      users = User.have_invoices().get()
      for user in users:
        # send invoice to user
        pass
```

# Registering Tasks

You must then register tasks to the Masonite scheduler. You can do so easily in a service provider:

```python
from app.tasks.SendInvoices import SendInvoices

def register(self):

  self.application.make('scheduler').add(
    SendInvoices().every_day()
  )
```

Tasks will run as often as you specify them to run using the time options.

# Options

You can specify the tasks to run as often as you need them to. Available options are:

| Option                   | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `every_minute()`         | Specifies this task to run every minute                      |
| `every_15_minutes()`     | Specifies this task to run every 15 minutes                  |
| `every_30_minutes()`     | Specifies this task to run every 30 minutes                  |
| `every_45_minutes()`     | Specifies this task to run every 45 minutes                  |
| `hourly()`               | Specifies this task to run every hour.                       |
| `daily()`                | Specifies this task to run every day at midnight             |
| `weekly()`               | Specifies this task to run every week on sunday at 00:00     |
| `monthly()`              | Specifies this task to run every first of the month at 00:00 |
| `at(17)`                 | Specifies the time to run the job at. Can be used with other options like `daily()` |
| `run_every('7 minutes')` | Specifies the amount of time to run. Can be any combination of time like `7 months`, `4 days`, `3 weeks`. |
| `daily_at(17)`           | Runs every day at the specified time. Time is in 24-hour time. `8` is "8 am" and `17` is "5pm". |
| `at_twice([8,17])`       | Runs at 8am and 5pm.                                         |

# Running The Tasks

To run all the tasks that are registered, we can find and execute the ones that should run depending on the time of the computer / server:

```terminal
$ python craft schedule:run
```

## Cron Jobs

> Setting up Cron Jobs are for UNIX based machines like Mac and Linux only. Windows has a similar schedule called Task Scheduler which is similar but will require different instructions in setting it up.

Although the command to `schedule:run` above is useful and needed, we still need a way to run it every minute. We can do this with a cronjob on the server

We'll show you an example cron job and then we will walk through how to build it.

```text
PATH=/Users/Masonite/Programming/project_name/venv/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Frameworks/Python.framework/Versions/3.6/bin

* * * * * cd /Users/Masonite/Programming/project_name && source venv/bin/activate && python craft schedule:run
```

### Getting The Path

When a cron job runs, it will typically run commands with a /bin/sh command instead of the usual /bin/bash. Because of this, craft may not be found on the machine so we need to tell the cron job the PATH that should be loaded in. We can simply find the PATH by going to our project directory and running:

```terminal
$ env
```

Which will show an output of something like:

```terminal
...
__CF_USER_TEXT_ENCODING=0x1F5:0x0:0x0
PATH=/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Frameworks/Python.framework/Versions/3.6/bin
PWD=/Users/Masonite/Programming/masonite
...
```

> If you are using a virtual environment for development purposes then you need to run the `env` command inside your virtual environment.

We can then copy the PATH and put it in the cron job.

To enter into cron, just run:

```terminal
$ env EDITOR=nano crontab -eqcszae3zd4rfdsxs
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