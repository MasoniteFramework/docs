# Introduction

Logging is a pretty crucial part of any application. The Masonite Logging package allows you to see errors your application is throwing as well as allow you to log your own messages in several different alert levels.

Masonite Logging currently contains the ability to log to a file, syslog and slack.

# Installation

To get the Masonite Logging package up and running on your machine you must first install the package:

```terminal
$ pip install masonite-logging
```

And then add the Service Provider to your application:

```python
from masonite.logging.providers import LoggingProvider
# ...
PROVIDERS = [
    # ..

    # Third Party Providers
    LoggingProvider,
    #..
]
```

You can then publish the provider which will create your `config/logging.py` configuration file:

```
$ craft publish LoggingProvider
```

# Logging Exceptions

The Masonite Logging package will automatically register an exception listener with your Masonite application and log any exceptions your application encounters with the corresponding channel.

# Channels

The Masonite Logging package uses the concept of channels and drivers. A channel internally is used to create and pass various information to instantiate the driver. At a higher level, you will mainly be working with channels.

Out of the box there are several different channels: `single`, `stack`, `daily`, `slack`, `syslog` and `terminal`. Each channel will handle logging messages in it's own way.

## Single Channel

The single channel will put all information in a "single" file. This file can be specified in your `config/logging.py` file in the `path` option:

```python
'single': {
    # ...
    'path': 'storage/logs/single.log'
},
```

## Daily Channel

The daily channel is similiar to the single channel except this will create a new log file based on the current day. So this will create log files like `10-23-2019.log` and `10-24-2019.log`. 

The path to set here needs to be a directory instead of a path to a file:

```python
'single': {
    # ...
    'path': 'storage/logs'
},
```

## Slack Channel

The Slack channel will send messages directly to your Slack channel so you or your team can act quickly and be alerted directly of any messages logged.

You'll need to generate a Slack application token. You can add it to your `.env` file:

```
SLACK_TOKEN=xoxp-35992....
```

You then need to set a few options in your config file if you need to change any default settings like the user, the icon emoji, 

```python
'slack': {
    # ...
    'channel': '#bot',
    'emoji': ':warning:',
    'username': 'Logging Bot',
    'token': env('SLACK_TOKEN', None),
    # ...
}
```

These options are up to you.

## Terminal Channel

The terminal channel will simply output errors to the terminal. This is handy for debugging or in addition to other channels when using the `stack` channel.

## Stack Channel

The stack channel is useful when you need to combine several channels together. Maybe you want it to log to both a `daily` channel file as well as send a message to your slack group. You can do this easily by specifying the `channels` within your `stack` channel options.

```python
'stack': {
    # ...
    'channels': ['daily', 'slack']
},
```

You can have as many channels as you want here.

## Syslog

The `syslog` channel will tie directly into your system level logging software. Each operating system has its own type of system monitoring.

You'll need to tie into your system socket path. This can be different per operating system and machine so find yours and put that socket path in the config options:

```python
'syslog': {
    # ...
    'path': '/var/run/syslog',
}
```

# Log Levels

Log levels are a hierarchy of log levels that you can specify on your channels. The hierarchy in order of most important to least important are:

```
emergency
alert
critical
error
warning
notice
info
debug
```

Each channel can have its own minimum log level. The log message will only continue if it is greater than or equal to the minimum log level on that channel.

For example, if we have a configuration like this on the `daily` channel:


```python
'daily': {
    'level': 'info',
    'path': 'storage/logs'
},
```

This will only send messages to this channel if the log level is notice or above. It will ignore all `debug` level log messages.

# Writing Log Messages

You can of course write your own log messages. You can resolve the logger class from the container and use a method equal to the log level you want to write the message for.

```python
from masonite.logging import Logger

def show(self, logger: Logger):
    logger.debug('message')
    logger.info('message')
    logger.notice('message')
    logger.warning('message')
    # ...
```

## Write to a specific channel

You can easily switch channels by using the `channel` method:

```python
from masonite.logging import Logger

def show(self, logger: Logger):
    logger.channel('slack').debug('message')
    # ...
```

# Timezones

By default, Masonite Logging will record all times in the `UTC` timezone but in the event you want to switch timezones, you can do so in your configuration file:

```python
CHANNELS = {
    'timezone': 'America/New_York',
    'single': {
        # ...
    },
```

All timestamps associated with logging will now use the correct timezone.
