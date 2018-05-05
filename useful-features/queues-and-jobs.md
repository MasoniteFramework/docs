# Queues and Jobs

## Introduction

Almost all applications can make use of queues. Queues are a great way to make time intensive tasks immediate by sending the task into the background. It's great to send anything and everything into the queue that doesn't require an immediate return value -- such as sending an email or firing an API call. The queue system is loaded into masonite via the `QueueProvider` Service Provider.

## Getting Started

All configuration settings by default are in the `config/queue.py` file. Out of the box, Masonite only supports the `async` driver which simply sends jobs into the background using multithreading. You are free to create more drivers. If you do create a driver, consider making it available on PyPi so others can also install it.

### Jobs

Jobs are simply Python classes that inherit the `Queueable` class that is provided by Masonite. We can simply create jobs using the `craft job` command.

```text
$ craft job SendWelcomeEmail
```

This will create a new job inside `app/jobs/SendWelcomeEmail.py`. Our job will look like:

```python
from masonite.queues.Queueable import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self):
        pass

    def handle(self):
        pass
```

All job constructors are resolved by the container so we can simply pass anything we need as normal:

```python
from masonite.queues.Queueable import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self, Request, Mail):
        self.request = Request
        self.mail = Mail

    def handle(self):
        pass
```

Remember that anything that is resolved by the container is able to retrieve anything from the container by simply passing in parameters of objects that are located in the container. Read more about the container in the [Service Container](../architectural-concepts/service-container.md) documentation.

Whenever jobs are executed, it simply executes the handle method. Because of this we can send our welcome email:

```python
from masonite.queues.Queueable import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self, Request, Mail):
        self.request = Request
        self.mail = Mail

    def handle(self):
        self.mail.driver('mailgun').to(self.request.user().email).template('mail/welcome').send()
```

That's it! We just created a job that can send to to the queue!

### Running Jobs

We can run jobs by using the `Queue` alias from the container. Let's run this job from a controller method:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail

def show(self, Queue):
    Queue.push(SendWelcomeEmail)
```

That's it! This job will be loaded into the queue. By default, Masonite uses the `async` driver which just sends tasks into the background.

We can also send multiple jobs to the queue by passing more of them into the `.push()` method:

```python
from app.jobs.SendWelcomeEmail import SendWelcomeEmail
from app.jobs.TutorialEmail import TutorialEmail

def show(self, Queue):
    Queue.push(SendWelcomeEmail, TutorialEmail)
```

