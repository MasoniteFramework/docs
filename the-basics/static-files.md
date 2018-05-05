# Static Files

## Introduction

Masonite tries to make static files extremely easy and comes with whitenoise out of the box. White noise wraps the WSGI application and listens for certain URI requests that can be resistered in your configuration files.

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

Thats it! Static files are extremely simple. You are now a master at static files!

