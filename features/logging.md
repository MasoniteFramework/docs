Logging is a pretty crucial part of any application. Logging allows you to see errors your application is throwing as well as allow you to log your own messages in several different alert levels.

Masonite Logging currently contains the ability to log to a file, syslog and slack.

# Configuration

To enable logging in your application, ensure that `LoggingProvider` is added to your application
providers:
```python
# config/providers.py
from masonite.providers import LoggingProvider

PROVIDERS = [
  # ...
  LoggingProvider
]
```

