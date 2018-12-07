# Hash ID's

## Introduction

The Hash ID Masonite essential feature is design for you to easily be able to screen your ID's in your URL's while being able to automatically decode them before they reach your controller method.

For example we can encode a value like `10` and get back `l9avmeG`. When we pass this value into our endpoint like `/dashboard/user/l9avmeG`, we get back the correct value in our controller:

```python
from masonite.request import Request

class SomeController:
    
    # Route: /dashboard/user/@id
    # URL: /dashboard/user/l9avmeG
    def show(self, request: Request):
        request.input('id') #== 10
```

This is very useful for hiding primary ID's and this package makes it very simple to do so.

## Installation

This package requires:

* Masonite 2.0+

You can install the required dependencies for the Hash ID feature by running:

```bash
$ pip install masonite-essentials[hashids]
```

## Configuration

Once you have it installed you can then add a middleware and a Service Provider

### Middleware

You can add the middleware in your `config/middleware.py` file:

```python
...
from masonite.contrib.essentials.middleware import HashIDMiddleware
...
HTTP_MIDDLEWARE = [
    ...
    MaintenanceModeMiddleware,
    HashIDMiddleware,
]
```

### Service Provider

You also need to add the Service Provider as well:

```python
from masonite.contrib.essentials.providers import HashIDProvider

PROVIDERS = [
    # Framework Providers
    AppProvider,
    SessionProvider,
    ..
    ..
    # Third Party Providers
    HashIDProvider,
    ..
    ..
]
```

Great. Once you do those 2 things you are ready to get started.

## Usage

### Views

In your views you can use the Hash ID template helper which was added when you added the Service Provider. We can use this like so:

```markup
{% for user in users %}
    <a href="/dashboard/user/{{ hashid(user.id) }}">User</a>
    <!-- Generates /dashboard/user/l9avmeG -->
{% endfor %}
```

### Controllers

The usage of this is feature in a controller is designed to be transparent. When the middleware method runs it will check all the request inputs and try to decode them. It will insert the decoded values back into the request input and ignore anything it could not correctly decode.

You can get the inputs like normal:

```python
from masonite.request import Request

class SomeController:
    
    # Route: /dashboard/user/@id
    # URL: /dashboard/user/l9avmeG
    def show(self, request: Request):
        request.param('id') #== 10
```

### Inputs

This also works for inputs so if you have an endpoint like `/dashboard/user?id=l9avmeG` then you can still get the correct value by using `input()`.

```python
from masonite.request import Request

class SomeController:
    
    # Route: /dashboard/user
    # URL: /dashboard/user?id=l9avmeG
    def show(self, request: Request):
        request.input('id') #== 10
```

### Encoding

You can also encode values yourself by importing the essentials helper:

```python
from masonite.request import Request
from masonite.contrib.essentials.helpers import hashid

class SomeController:
    def show(self):
        hashid(10) #== l9avmeG
```

### Decoding

You have a few options for decoding. You can decode a normal string that you previously encoded by passing in the `decode` keyword.

```python
from masonite.contrib.essentials.helpers import hashid
​
class SomeController:
    def show(self):
        hashid('l9avmeG', decode=True) #== l9avmeG
```

You can also pass in a dictionary of values to decode. This helper will try to decode each one and insert the correct value. It will skip the ones it cannot decode.

```python
from masonite.contrib.essentials.helpers import hashid
​
class SomeController:
    def show(self):
        values = {'id': 'l9avmeG': 'name': 'Joe'}
        hashid(values, decode=True) #== {'id': '10': 'name': 'Joe'}
```

This is actually what is happening under the hood of the feature.

