# Static Files

## Introduction

Masonite tries to make static files extremely easy and comes with white noise out of the box. White noise wraps the WSGI application and listens for certain URI requests that can be resistered in your configuration files.

## Configuration

All configurations that are specific to static files can be found in `config/storage.py`. In this file you'll find a constant file called `STATICFILES` which is simply a dictionary of directories as keys and aliases as the value.

The directories to include as keys is simply the location of your static file locations. For example, if your css files are in `storage/assets/css` then put that folder location as the key. For the value, put the alias you want to use in your templates. For this example, we will use `css/` as the alias.

For this setup, our `STATICFILES` constant should look like:

```python
STATICFILES = {
    'storage/assets/css': 'css/',
}
```

Now in our templates we can use:

```markup
<img src="/css/style.css">
```

Which will get the `storage/assets/css/style.css` file.

## Static Template Function

All templates have a static function that can be used to assist in getting locations of static files. You can specify the driver and locations you want using the driver name or dot notation.

Take this for example:

```python
....
's3': {
  's3_client': 'sIS8shn...'
  ...
  'location': 'https://s3.us-east-2.amazonaws.com/bucket'
  },
....
```

```markup
...
<img src="{{ static('s3', 'profile.jpg') }}" alt="profile">
...
```

this will render:

```python
<img src="https://s3.us-east-2.amazonaws.com/bucket/profile.jpg" alt="profile">
```

You can also make the config location a dictionary and use dot notation:

```python
....
's3': {
  's3_client': 'sIS8shn...'
  ...
  'location': {
    'east': 'https://s3.us-east-2.amazonaws.com/east-bucket',
    'west': 'https://s3.us-west-16.amazonaws.com/west-bucket'
  },
....
```

and use the dot notation like so:

```markup
...
<img src="{{ static('s3.east', 'profile.jpg') }}" alt="profile">
...
<img src="{{ static('s3.west', 'profile.jpg') }}" alt="profile">
...
```

That's it! Static files are extremely simple. You are now a master at static files!

