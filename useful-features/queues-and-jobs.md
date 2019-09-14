# Queues and Jobs

## Queues and Jobs

### Introduction

Almost all applications can make use of queues. Queues are a great way to make time intensive tasks seem immediate by sending the task into the background or into a message queue. It's great to send anything and everything into the queue that doesn't require an immediate return value \(such as sending an email or firing an API call\). The queue system is loaded into masonite via the `QueueProvider` Service Provider.

{% hint style="info" %}
Masonite uses pickle to serialize and deserialize Python objects when appropriate. Ensure that the objects you are serializing is free of any end user supplied code that could potentially serialize into a Python object during the deserialization portion.

It would be wise to read about [pickle exploitations](https://blog.nelhage.com/2011/03/exploiting-pickle/) and ensure your specific application is protected against any avenues of attack.
{% endhint %}

### Getting Started

All configuration settings by default are in the `config/queue.py` file. Out of the box, Masonite supports 3 drivers:

* `async`
* `amqp`
* `database`

The `async` driver simply sends jobs into the background using multithreading. The `amqp` driver is used for any AMQP compatible message queues like RabbitMQ. If you do create a driver, consider making it available on PyPi so others can also install it. The `database` driver has a few additional features that the other drivers do not have if you need more fine-grained control 

#### Jobs

Jobs are simply Python classes that inherit the `Queueable` class that is provided by Masonite. We can simply create jobs using the `craft job` command.

```text
$ craft job SendWelcomeEmail
```

This will create a new job inside `app/jobs/SendWelcomeEmail.py`. Our job will look like:

```python
from masonite.queues import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self):
        pass

    def handle(self):
        pass
```

#### Running Jobs

We can run jobs by using the `Queue` class. Let's run this job from a controller method:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail
from masonite import Queue

def show(self, queue: Queue):
    queue.push(SendWelcomeEmail)
```

That's it. This job will now send to the queue and run the `handle` method.

### Resolving

Notice in the show method above that we passed in just the class object. We did not instantiate the class. In this instance, Masonite will resolve the controller constructor. All job constructors are able to be resolved by the container so we can simply pass anything we need as normal:

```python
from masonite.queues import Queueable
from masonite.request import Request
from masonite import Mail

class SendWelcomeEmail(Queueable):

    def __init__(self, request: Request, mail: Mail):
        self.request = request
        self.mail = mail

    def handle(self):
        pass
```

Remember that anything that is resolved by the container is able to retrieve anything from the container by simply passing in parameters of objects that are located in the container. Read more about the container in the [Service Container](../architectural-concepts/service-container.md) documentation.

### Instantiating

We can also instantiate the job as well if we need to pass in data from a controller method. This will not resolve the job's constructor at all:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail
from masonite import Queue

def show(self, queue: Queue):
    var1 = 'value1'
    var2 = 'value2'

    queue.push(SendWelcomeEmail(var1, var2))
```

The constructor of our job class now will look like:

```python
class SendWelcomeEmail(Queueable):

    def __init__(self, var1, var2):
        self.var1 = var1
        self.var2 = var2
```

### Executing Jobs

Whenever jobs are executed, it simply executes the handle method. Because of this we can send our welcome email:

```python
from masonite.queues import Queueable
from masonite.request import Request
from masonite import Mail

class SendWelcomeEmail(Queueable):

    def __init__(self, request: Request, mail: Mail):
        self.request = request
        self.mail = mail

    def handle(self):
        self.mail.driver('mailgun').to(self.request.user().email).template('mail/welcome').send()
```

That's it! This job will be loaded into the queue. By default, Masonite uses the `async` driver which just sends tasks into the background.

We can also send multiple jobs to the queue by passing more of them into the `.push()` method:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail
from app.jobs.TutorialEmail import TutorialEmail
from masonite import Queue

def show(self, queue: Queue):
    queue.push(SendWelcomeEmail, TutorialEmail('val1', 'val2'))
```

### Passing Variables Into Jobs

Most of the time you will want to resolve the constructor but pass in variables into the `handle()` method. This can be done by passing in an iterator into the `args=` keyword argument:

```python
from masonite import Queue

def show(self, queue: Queue):
    queue.push(SendWelcomeEmail, args=['user@email.com'])
```

This will pass to your handle method:

```python
from masonite.request import Request
from masonite import Mail
class SendWelcomeEmail(Queueable):

    def __init__(self, request: Request, mail: Mail):
        self.request = request
        self.mail = mail

    def handle(self, email):
        email # =='user@email.com'
```

### Passing Functions or Methods

You can also call any arbitrary function or method using the queue driver. All you need to do is pass the reference for it in the push method and pass any arguments you need in the args parameter like so:

```python
def run_async(obj1, obj2):
    pass

def show(self, queue: Queue):
    obj1 = SomeObject()
    obj2 = AnotherObject()
    queue.push(run_async, args=(obj1, obj2))
```

This will then queue this function to be called later.

{% hint style="warning" %}
Note that you will not be able to get a response value back. Once it gets sent to the queue it will run at an arbitrary time later.
{% endhint %}

## Async Driver

The `async` queue driver will allow you to send jobs into the background to run asynchronously. This does not need any third party services like the `amqp` driver below.

### Change Modes

The async driver has 2 different modes: `threading` and `multiprocess`.  The differences between the two is that `threading` uses several threads and `multiprocess` uses several processes. Which mode you should use depends on the type of jobs you are processing. You should research what is best depending on your use cases.

You can change the mode inside the `config/queue.py` file:

```python
DRIVERS = {
    'async': {
        'mode': 'threading' # or 'multiprocess'
    },
}
```

### Blocking

During development it may be hard to debug asyncronous tasks. If an exception is thrown it will be hard to catch that. It may appear that a job is never ran.

In order to combat this you can set the `blocking` setting in your `config/queue.py` file:

```python
DRIVERS = {
    'async': {
        'mode': 'threading' # or 'multiprocess',
        'blocking': True
    },
}
```

Blocking bascially makes asyncronous tasks run syncronously. This will enable some reporting inside your terminal that looks something like:

```
GET Route: /categories
 Job Ran: <Future at 0x1032cef60 state=finished returned str> 
 Job Ran: <Future at 0x1032f1a90 state=finished returned str> 
 ...
```

This will also run tasks syncronously so you can find exceptions and issues in your jobs during development.

For production this should be set to `False`.

It may be good to set this setting equal to whatever your `APP_DEBUG` environment variable is:

```python
from masonite import env

DRIVERS = {
    'async': {
        'mode': 'threading' # or 'multiprocess',
        'blocking': env('APP_DEBUG')
    },
}
```

This way it will always be blocking during development and automatically switch to unblocking during production.

## AMQP Driver

The `amqp` driver can be used to communicate with RabbitMQ services.

### Installing

In order to get started with this driver you will need to install RabbitMQ on your development machine \(or production machine depending on where you are running Masonite\)

You can find the [installation guide for RabbitMQ here](https://www.rabbitmq.com/download.html).

### Running RabbitMQ

Once you have RabbitMQ installed you can go ahead and run it. This looking something like this in the terminal if ran successfully:

```bash
$ rabbitmq-server

  ##  ##
  ##  ##      RabbitMQ 3.7.8. Copyright (C) 2007-2018 Pivotal Software, Inc.
  ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
  ######  ##
  ##########  Logs: /usr/local/var/log/rabbitmq/rabbit@localhost.log
                    /usr/local/var/log/rabbitmq/rabbit@localhost_upgrade.log

              Starting broker...
 completed with 6 plugins.
```

Great! Now that RabbitMQ is up and running we can look at the Masonite part.

Now we will need to make sure our driver and driver configurations are specified correctly. Below are the default values which should connect to your current RabbitMQ configuration. This will be in your `app/queue.py` file

```python
DRIVER = 'amqp'
...
DRIVERS = {
    'amqp': {
        'username': 'guest',
        'password': 'guest',
        'host': 'localhost',
        'port': '5672',
        'channel': 'default',
    }
}
```

If your rabbit MQ instance requires a `vhost` but doesn't have a port, we can add a `vhost` and set the port to none. `vhost` and `port` both have the option of being `None`. If you are developing locally then `vhost` should likely be left out all together. The setting below will most likely be used for your production settings:

```python
DRIVER = 'amqp'
...
DRIVERS = {
    'amqp': {
        'username': 'guest',
        'vhost': '/',
        'password': 'guest',
        'host': 'localhost',
        'port': None,
        'channel': 'default',
    }
}
```

## Database Driver

The database driver will store all jobs in a database table called `queue_jobs` and on fail, will store all failed jobs in a `failed_jobs` table if one exists. If the `failed_jobs` table does not exist then it will not store any failed jobs and any jobs that fail will be lost.

### Migrations

In order to get these two queue table you can run the `queue:table` command with the flag on which table you would like:

This command will create the `queue_jobs` migration where you can store your jobs:

```
$ craft queue:table --jobs
```

This command will create the `failed_jobs` migration where you can store your failed jobs:

```
$ craft queue:table --failed
```

Once these migrations are created you can run the migrate command:

```
$ craft migrate
```

### Delaying Jobs

Jobs can be easily delayed using the `database` driver. Other drivers currently do not have this ability. In order to delay a job you can use a string time using the `wait` keyword.

```python
def show(self, queue: Queue):
    queue.push(SendWelcomeEmail, wait="10 minutes")
```

### Starting The Worker

We can now start the worker using the `queue:work` command. It might be a good idea to run this command in a new terminal window since it will stay running until we close it.

```bash
$ craft queue:work
```

This will startup the worker and start listening for jobs to come in via your Masonite project.

You can also specify the driver you want to create the worker for by using the `-d` or `--driver` option

```bash
$ craft queue:work --driver amqp
```

You may also specify the `channel` as well. `channel` may mean different things to different drivers. For the `amqp` driver, the `channel` is which queue to listen to. For the `database` driver, the `channel` is the connection to find the `queue_jobs` and `queue_failed` tables.

```bash
$ craft queue:work --driver database --channel sqlite
```

### Sending Jobs

That's it! send jobs like you normally would and it will process via RabbitMQ:

```python
from app.jobs import SomeJob, AnotherJob
from masonite import Queue
...
def show(self, queue: Queue):
    # do your normal logic
    queue.push(SomeJob, AnotherJob(1,2))
```

you can also specify the channel to push to by running:

```python
queue.push(SomeJob, AnotherJob(1,2), channel="high")
```

## Failed Jobs

Sometimes your jobs will fail. This could be for many reasons such as an exception but Masonite will try to run the job 3 times in a row, waiting 1 second between jobs before finally calling the job failed.

If the object being passed into the queue is not a job \(or a class that implements `Queueable`\) then the job will not requeue. It will only ever attempt to run once.

### Handling Failed Jobs

Each job can have a `failed` method which will be called when the job fails. You can do things like fix a parameter and requeue something, call other queues, send an email to your development team etc.

This will look something like:

```python
from masonite.queues import Queueable
from masonite.request import Request
from masonite import Mail

class SendWelcomeEmail(Queueable):

    def __init__(self, request: Request, mail: Mail):
        self.request = request
        self.mail = mail

    def failed(self, payload, error):
        self.mail.to('developer@company.com').send('The welcome email failed')
```

It's important to note that only classes that extend from the `Queueable` class will handle being failed. All other queued objects will simply die with no failed callback.

Notice that the failed method MUST take 2 parameters.

The first parameter is the payload which tried running which is a dictionary of information that looks like this:

```python
payload == {
    'obj': <class app.jobs.SomeJob>,
    'args': ('some_variables',), 
    'callback': 'handle', 
    'created': '2019-02-08T18:49:59.588474-05:00', 
    'ran': 3
}
```

and the error may be something like `division by zero`.

### Storing Failed Jobs

By default, when a job is failed it disappears and cannot be ran again since Masonite does not store this information.

If you wish to store failed jobs in order to run them again at a later date then you will need to create a queue table. Masonite makes this very easy.

First you will need to run:

```text
$ craft queue:table
```

Which will create a new migration inside `databases/migrations`. Then you can will migrate it:

```text
$ craft migrate
```

Now whenever a failed job occurs it will store the information inside this new table.

### Running Failed Jobs

You can run all the failed jobs by running

```text
$ craft queue:work --failed
```

This will get all the jobs from the database and send them back into the queue. If they fail again then they will be added back into this database table.

## Specifying Failed Jobs

You can modify the settings above by specifying it directly on the job. For example you may want to specify that the job reruns 5 times instead of 3 times when it fails or that it should not rerun at all.

Specifying this on a job may look something like:

```python
from masonite.request import Request
from masonite import Mail

class SendWelcomeEmail(Queueable):

    run_again_on_fail = False

    def __init__(self, request: Request, mail: Mail):
        self.request = Request
        self.mail = Mail

    def handle(self, email):
        ...
```

This will not try to rerun when the job fails.

You can specify how many times the job will rerun when it fails by specifying the `run_times` attribute:

```python
from masonite.request import Request
from masonite import Mail

class SendWelcomeEmail(Queueable):

    run_times = 5

    def __init__(self, request: Request, mail: Mail):
        self.request = Request
        self.mail = Mail

    def handle(self, email):
        ...
```

