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
        self.mail = Mail

    def handle(self):
        pass
```

