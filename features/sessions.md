Masonite comes with a simple way to store sessions. Masonite supports the following session drivers: [Cookie](#cookie) and [Redis](#redis).

# Configuration

Session configuration is located at `config/session.py` file. In this file, you can configure which driver
to use.

Masonite is configured to use the Cookie session driver by default, named `cookie`.

{% code title="config/session.py" %}
```python
DRIVERS = {
    "default": "cookie",
    "cookie": {},
    "redis": {
        "host": "127.0.0.1",
        "port": 6379,
        "password": "",
        "options": {"db": 1},  # redis module driver specific options
        "timeout": 60 * 60,
        "namespace": "masonite4",
    },
}
```
{% endcode %}

## Cookie

Cookie driver will store all session data in the users cookies. It can be used as is.

## Redis

Redis driver is requiring the `redis` python package, that you can install with:
```bash
pip install redis
```

Then you should define Redis as default driver and configure it with your Redis server parameters:
```python
DRIVERS = {
    "default": "redis",
    "redis": {
        "host": "127.0.0.1",
        "port": 6379,
        "password": "",
        "options": {"db": 1},  # redis module driver specific options
        "timeout": 60 * 60,
        "namespace": "masonite4",
    },
}
```

Finally ensure that the Redis server is running and you're ready to start using sessions.

# Saving Session Data

To save session data you can simply "set" data into the session:

```python
from masonite.sessions import Session

def store(self, session: Session):
  data = session.set('key', 'value')
```

# Flashing Data

Flash data is data specific to the next request. This is useful for data such as error messages or alerts:

```python
from masonite.sessions import Session

def store(self, session: Session):
  data = session.flash('key', 'value')
```

# Retrieving Session Data

To get back the session data you set you can simply using the "get" method:

```python
from masonite.sessions import Session

def store(self, session: Session):
  data = session.get('key')
```

# Checking For Existence

You can check if a session has a specific key:

```python
from masonite.sessions import Session

def store(self, session: Session):
  if session.has('key'):
    pass
```

# Deleting Session Data

You can also delete a key from the session

```python
from masonite.sessions import Session

def store(self, session: Session):
  session.delete('key')
```

# Resetting the Session

You can reset all data in a session:

```python
session.flush()
```
