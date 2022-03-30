Masonite comes with a simple way to store sessions. Currently the only session driver available is the `cookie` driver which will store all session data in the users cookies.

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
