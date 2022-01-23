# Masonite Debugbar

## Introduction

Debugging and monitoring application performance should be made easy to Masonite developers. The Masonite Debugpar package will add an easy to use debugbar at the bottom of your application in your local environment.

## Installation

```text
$ pip install masonite-debugbar
```

And then add the `DebugProvider` provider to your application:

```python
from debugbar.providers import DebugProvider

PROVIDERS = [
    # ..
    DebugProvider,

]
```

Finally you can publish package files to customize it:

```text
$ python craft package:publish debugbar
```
