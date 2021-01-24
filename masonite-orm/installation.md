# Installation

Setting up Masonite is extremely simple.

If you are using the Masonite web framework than all the installation is setup for you. If you are using anything other than Masonite or building your own Python application then be sure to follow the install steps below:

## Pip install

First install via pip:

```text
$ pip install masonite-orm
```

## Configuration File

To start configuring your project you'll need a `config/database.py` file. In this file we will be able to put all our connection information.

One we have our `config/database.py` file we can put a `DATABASES` variable with a dictionary of connection details. Each key will be the name of our connection. The connection name can be whatever you like and does not need to relate to a database name. Common connection names could be something like `dev`, `prod` and `staging`. Feel free to name these connections whatever you like.

The connection variable will look something like this

```python
# config/database.py
DATABASES = {
  "default": "mysql",
  "mysql": {
    "host": "127.0.0.1",
    "database": "masonite",
    "user": "root",
    "password": "",
    "port": 3306
    "prefix": "",
    "options": {
      #  
    }
  },  
  "postgres": {
    "host": "127.0.0.1",
    "database": "masonite",
    "user": "root",
    "password": "",
    "port": 5432
    "prefix": "",
    "options": {
      #  
    }
  },
  "sqlite": {
    "database": "masonite.sqlite3",
  }
}
```

Lastly you will need to import the `ConnectionResolver` class and and register the connection details:

```python
# config/database.py
from masoniteorm.connections import ConnectionResolver

DATABASES = {
  # ...
}

ConnectionResolver().set_connection_details(DATABASES)
```

After this you have successfully setup Masonite ORM in your project!

## Logging

If you would like, you can log any queries Masonite ORM generates to any supported Python logging handler. 

Inside your `config/database.py` file you can put on the bottom here. The StreamHandler will output the queries to the terminal.

```python
logger = logging.getLogger('masoniteorm.connection.queries')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()

logger.addHandler(handler)
```

You can specify as many handlers as you would like. Here's an example of logging to both the terminal and a file:

```python
logger = logging.getLogger('masoniteorm.connection.queries')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
file_handler = logging.FileHandler('queries.log')

logger.addHandler(handler)
logger.addHandler(file_handler)
```