# Queues and Jobs

Masonite ships with a powerful queue system. This feature is useful for running those tasks that take a while like sending emails, processing videos, creating invoices, updating records and anything else you don't need you users to wait for or jobs that need .

First, jobs are creating with the logic required to make the job run. The jobs are then "pushed" onto the queue where they will run later using "queue workers". You can specify as many queue workers as your server can run.

In addition to running jobs, some drivers allow you to monitor any jobs that fail. Job data will save into the database where they can be monitored and reran if needed.

## Configuration

You can easily modify the behavior of your queue system by changing the queue configuration:

The available queue drivers are: `async`, `database` and `amqp`.

A full queue configuration will look like this:

```python
DRIVERS = {
    "default": "async",
    "database": {
        "connection": "mysql",
        "table": "jobs",
        "failed_table": "failed_jobs",
        "attempts": 3,
        "poll": 5,
        "tz": "UTC"
    },
    "amqp": {
        "username": "guest",
        "password": "guest",
        "port": "5672",
        "vhost": "",
        "host": "localhost",
        "channel": "default",
        "queue": "masonite4",
    },
    "async": {
        "blocking": False,
        "callback": "handle",
        "mode": "threading",
        "workers": 1,
    },
}
```

### Default Queue

The default key is used to specify which queue driver in the configuration to use by default. This needs to be the same value as one of the other keys in the configuration dictionary.

### Database Driver

To use the database driver you should first create a jobs table:

```
$ python craft queue:table
```

This will create a migration file in your migrations directory.

If you want to save failed jobs you should also create a migration for the failed jobs table:

```
$ python craft queue:failed
```

You should then migrate your project:

```
$ python craft migrate
```

The database driver is used to process jobs and failed job via a database connection.

| Option        | Description                                                                                |
| ------------- | ------------------------------------------------------------------------------------------ |
| `connection`  | Specifies the connection to use for finding the jobs and failed jobs table.                |
| `table`       | Specifies the name of the table to store jobs in                                           |
| `failed_jobs` | Specifies the table to store failed\_jobs in. Set to `None` to not save failed jobs.       |
| `attempts`    | Specifies the default number of times to attempt a job before considering it a failed job. |
| `poll`        | Specifies the time in seconds to wait before calling the database to find new jobs         |
| `tz`          | The timezone that the database should save and find timestamps in                          |

### AMQP Driver

The AMQP driver is used for connection that use the AMQP protocol, such as RabbitMQ.

The available options include:

| Option     | Description                                                                        |
| ---------- | ---------------------------------------------------------------------------------- |
| `username` | The username of your AMQP connection                                               |
| `password` | The password of your AMQP connection                                               |
| `port`     | The port of your AMQP connection                                                   |
| `vhost`    | The name of your virtual host. Can get this through your AMQP connection dashboard |
| `host`     | The IP address or host name of your connection.                                    |
| `channel`  | The channel to push the queue jobs onto                                            |
| `queue`    | The default name of the queue to push the jobs onto.                               |

### Async Driver

The async driver will simply run the jobs in memory using processes or threading. This is the simplest driver as it does not need any special software or setup.

The available options include:

| Option     | Description                                                                                         |
| ---------- | --------------------------------------------------------------------------------------------------- |
| `blocking` | <p>A boolean value on whether jobs should run synchronously. <br>Useful for debugging purposes.</p> |
| `callback` | The name of the method on the job that should run.                                                  |
| `mode`     | Whether the queue should spawn processes or threads. Options are `threading` or `multiprocess`      |
| `workers`  | The numbers of processes or threads that should spawn to run  the jobs.                             |

## Creating Jobs

In order to process things on the queue, you will need to create a job. This job will be treated as an entity that can be serialized and ran later.

For a shortcut you can run the job command to create the job:

```
$ python craft job CreateInvoice
```

You will now have a job class you can build out the logic for:

```python
from masonite.queues import Queueable

class CreateInvoice(Queueable):
    def handle(self):
        pass
```

Any logic should be inside the handle method:

```python
class CreateInvoice(Queueable):

    def __init__(self, order_id):
        self.order_id = order_id

    def handle(self):
        # Generate invoice documents
        pass
```

## Queueing Jobs

You can put jobs on the queue to process by simply passing them onto the queue:

```python
from masonite.queues import Queue
from app.jobs.CreateInvoice import CreateInvoice

class InvoiceController:

  def generate(self, queue: Queue):
    # Jobs with no parameters
    queue.push(CreateInvoice())

    # Jobs with parameters
    # Create an invoice from a payment
    queue.push(CreateInvoice(Order.find(1).id))
```

You can also specify any number of options using keyword arguments on the push method:

```python
queue.push(
  CreateInvoice(Order.find(1).id),
  driver="async", # The queue driver to use
  queue="invoices", # The queue name to put the job on
)
```

## Queue Workers

To run a queue worker, which is a terminal process than runs the jobs, you can use the `queue:work` command:

```
$ python craft queue:work
```

This will start up a worker using the default queue configurations. You can also modify the options:

| Option               | Description                                                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------- |
| `--driver database`  | Specifies which driver to use for this worker.                                                          |
| `--queue invoices`   | Specifis which queue to use for this worker.                                                            |
| `--connection mysql` | Specifies the connection to use for the worker.                                                         |
| `--poll 5`           | Specifies the time in seconds to wait to fetch new jobs. Default is 1 second.                           |
| `--attempts 5`       | Specifies the number of attempts to retry a job before considering it a failed job. Default is 3 times. |

A command with modified options will look like this:

```
$ python craft queue:work --driver database --connection mysql --poll 5 --attempts 2
```

## Failed Jobs

If you configurations are setup properly, when jobs fail, they will go into a failed jobs table. This is where you can monitor why your jobs are failing and choose to rerun them or remove them.

If you choose to rerun your jobs, they will be placed back onto the queue at the end and rerun with the normal job queuing process.

To rerun jobs that failed you can use the command:

```
$ python craft queue:retry
```

You can specify a few options as well:

| Option               | Description                                                           |
| -------------------- | --------------------------------------------------------------------- |
| `--driver database`  | Specifies which driver to use to find the failed jobs.                |
| `--queue invoices`   | Specifis which queue to put the failed jobs back onto the queue with. |
| `--connection mysql` | Specifies the connection to use to fetch the failed jobs.             |
