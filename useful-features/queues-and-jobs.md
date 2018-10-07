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

