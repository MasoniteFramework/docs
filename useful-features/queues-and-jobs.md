# Queues and Jobs

## Introduction

Almost all applications can make use of queues. Queues are a great way to make time intensive tasks seem immediate by sending the task into the background or into a message queue. It's great to send anything and everything into the queue that doesn't require an immediate return value (such as sending an email or firing an API call). The queue system is loaded into masonite via the `QueueProvider` Service Provider.

## Getting Started

All configuration settings by default are in the `config/queue.py` file. Out of the box, Masonite supports 2 drivers:

* `async`
* `amqp`

The `async` driver simply sends jobs into the background using multithreading. The `amqp` driver is used for any AMQP compatible message queues like RabbitMQ. If you do create a driver, consider making it available on PyPi so others can also install it.

### Jobs

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

### Running Jobs

We can run jobs by using the `Queue` alias from the container. Let's run this job from a controller method:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail

def show(self, Queue):
    Queue.push(SendWelcomeEmail)
```

## Resolving

Notice in the show method above that we passed in just the class object. We did not instantiate the class. In this instance, Masonite will resolve the controller constructor. All job constructors are able to be resolved by the container so we can simply pass anything we need as normal:

```python
from masonite.queues import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self, Request, Mail):
        self.request = Request
        self.mail = Mail

    def handle(self):
        pass
```

Remember that anything that is resolved by the container is able to retrieve anything from the container by simply passing in parameters of objects that are located in the container. Read more about the container in the [Service Container](../architectural-concepts/service-container.md) documentation.

## Instantiating

We can also instantiate as the job as well if we need to pass in data from a controller method. This will not resolve the job's constructor at all:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail

def show(self, Queue):
    var1 = 'value1'
    var2 = 'value2'
    
    Queue.push(SendWelcomeEmail(var1, var2))
```

The constructor of our job class now will look like:

```python
class SendWelcomeEmail(Queueable):

    def __init__(self, var1, var2):
        self.var1 = var1
        self.var2 = var2
```

## Executing Jobs

Whenever jobs are executed, it simply executes the handle method. Because of this we can send our welcome email:

```python
from masonite.queues import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self, Request, Mail):
        self.request = Request
        self.mail = Mail

    def handle(self):
        self.mail.driver('mailgun').to(self.request.user().email).template('mail/welcome').send()
```

That's it! We just created a job that can send to to the queue!

That's it! This job will be loaded into the queue. By default, Masonite uses the `async` driver which just sends tasks into the background.

We can also send multiple jobs to the queue by passing more of them into the `.push()` method:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail
from app.jobs.TutorialEmail import TutorialEmail

def show(self, Queue):
    Queue.push(SendWelcomeEmail, TutorialEmail('val1', 'val2'))
```

# AMQP Driver

The `amqp` driver can be used to communicate with RabbitMQ services.

## Installing

In order to get started with this driver you will need to install RabbitMQ on your development machine (or production machine depending on where you are running Masonite)

You can find the [installation guide for RabbitMQ here](https://www.rabbitmq.com/download.html).

## Running RabbitMQ

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

## Starting The Worker

We can now start the worker using the `queue:work` command. It might be a good idea to run this command in a new terminal window since it will stay running until we close it.

```bash
$ craft queue:work
```

This will startup the worker and start listening jobs to come in via your RabbitMQ instance.

## Sending Jobs

That's it! send jobs like you normally would and it will process via RabbitMQ:

```python
from app.jobs import SomeJob, AnotherJob
...
def show(self, Queue):
    # do your normal logic
    Queue.push(SomeJob, AnotherJob(1,2))
```