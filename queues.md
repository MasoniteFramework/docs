# Queues

# Introduction

Almost all applications can make use of queues. Queues are a great way to make time intensive tasks immediate by sending the task into the background. 

## Getting Started

All configuration settings by default are in the `config/queue.py` file. Out of the box, Masonite only supports the `async` driver which simply sends jobs into the background using multithreading.

## Jobs

Jobs are simply Python classes that inherit the `Queueable` class that is provided by Masonite. We can simply create jobs using the `craft job` command.

```
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

Remember that anything that is resolved by the container is able to retrieve anything from the container by simply passing in parameters of objects that are located in the container. Read more about the container in the "Service Container" documentation.

Whenever jobs are executed, it simply executes the handle method. Because of this we can send our welcome email:

```python
from masonite.queues.Queueable import Queueable

class SendWelcomeEmail(Queueable):

    def __init__(self, Request, Mail):
        self.request = Request
        self.mail = Mail

    def handle(self):
        self.mail.driver('smtp').to(
            self.request.user().email
        ).template('mail/welcome').send()
```


