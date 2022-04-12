Facades are an easy way to access the [Service Container](../architecture/service-container.md) classes without making them from the [Service Container](../architecture/service-container.md).

# Overview
Facades are just a shortcut to resolve a key in the [Service Container](../architecture/service-container.md). Instead of doing:

```python
application.make("mail").send()
```

you can do:

```python
from masonite.facades import Mail

Mail.send()
```

To import any built-in facades you can import them from the `masonite.facades` namespace:

```python
from masonite.facades import Request

def show(self):
    avatar = Request.input('avatar_url')
```

# Built-in Facades

Masonite ships with several facades you can use out of the box:

- `Auth`
- `Broadcast`
- `Config`
- `Dump`
- `Gate`
- `Hash`
- `Loader`
- `Mail`
- `Notification`
- `Queue`
- `Request`
- `Response`
- `Session`
- `Storage`
- `View`
- `Url`

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

To benefit from code intellisense/type hinting when using Facade (e.g. in VSCode editor) you should create a `.pyi` file just next to your facade file class to add type hinting to your new facade.

Then in this facade you should declare your typed methods. Here is a partial example of the Mail facades types hints:

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from ..mail import Mailable

class Mail:
    """Handle sending e-mails from Masonite via different drivers."""

    def mailable(mailable: "Mailable") -> "Mail":
        ...
    def send(driver: str = None):
        ...
```

Notice that:
  - methods code is not present, but replaced with `...`.
  - methods do not receive `self` as first argument. Because when calling the facade we don't really instantiate it (even if we get an instance of the object bound to the container). This allow to have the correct type hint behaviour.