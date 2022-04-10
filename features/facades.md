Facades are an easy way to access the container classes without making them from the container

# Built in Facades

Masonite ships with several facades you can use out of the box.

Facades are just a shortcut to a key in the IOC container.

To import a facade you can import them from the `masonite.facades` namespace:

```python
from masonite.facades import Request

def show(self):
  avatar = Request.input('avatar_url')
```

The available facades are:

- `Request`
- `Response`
- `View`
- `Mail`
- `Hash`
- `Session`
- `Notification`
- `Auth`
- `Config`
- `Url`
- `Broadcast`

# Creating Your Own Facades

To create your own facade is simple:

```python
from masonite.facades import Facade

class YourFacade(metaclass=Facade):
  key = 'container_key'
```

Then import and use your facade:

```python
from app.facades import YourFacade

YourFacade.method()
```
