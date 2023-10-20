Masonite tries to make static files extremely easy and comes with whitenoise out of the box. Whitenoise wraps the WSGI application and listens for certain URI requests that can be registered in your configuration files and serves those assets.

## Configuration

All configurations that are specific to static files can be found in `config/filesystem.py`. In this file you'll find a constant file called `STATICFILES` which is simply a dictionary of directories as keys and aliases as the value.

The directories to include as keys are simply the location of your static file locations as a relative path starting from the base of your application. For example, if your css files are in `storage/assets/css` then put that folder location as the key. For the value, put the alias you want to use in your templates. For this example, we will use `css/` as the alias.

For this setup, our `STATICFILES` constant should look like:

{% code title="config/filesystem.py" %}

```python
STATICFILES = {
    'storage/assets/css': 'assets/',
}
```

{% endcode %}

Now in our templates we can use:

```markup
<img src="/assets/style.css">
```

Which will get the `storage/assets/css/style.css` file.

## Static Template Function

All templates have a static function that can be used to assist in getting locations of static files. You can specify the driver and locations you want using the driver name or dot notation.

Take this for example:

{% code title="config/filesystem.py" %}

```python
....
's3': {
  's3_client': 'sIS8shn...'
  ...
  'path': 'https://s3.us-east-2.amazonaws.com/bucket'
  },
....
```

{% endcode %}

```html
...
<img src="{{ asset('s3', 'profile.jpg') }}" alt="profile" />
...
```

this will render:

```html
<img
  src="https://s3.us-east-2.amazonaws.com/bucket/profile.jpg"
  alt="profile"
/>
```

You can also make the config location a dictionary and use dot notation:

{% code title="config/filesystem.py" %}

```python
....
's3': {
  's3_client': 'sIS8shn...'
  ...
  'path': {
    'east': 'https://s3.us-east-2.amazonaws.com/east-bucket',
    'west': 'https://s3.us-west-16.amazonaws.com/west-bucket'
  },
....
```

{% endcode %}

and use the dot notation like so:

```html
...
<img src="{{ asset('s3.east', 'profile.jpg') }}" alt="profile" />
...
<img src="{{ asset('s3.west', 'profile.jpg') }}" alt="profile" />
...
```

## Serving "Root" Files

Sometimes you may need to serve files that are normally in the root of your application such as a `robots.txt` or `manifest.json`. These files can be aliased in your `STATICFILES` directory in `config/filesystem.py`. They do not have to be in the root of your project but instead could be in a `storage/root` or `storage/public` directory and aliased with a simple `/`.

For example a basic setup would have this as your directory:

```text
resources/
routes/
storage/
  static/
  root/
    robots.txt
    manifest.json
```

and you can alias this in your `STATICFILES` constant:

{% code title="config/filesystem.py" %}

```python
STATICFILES = {
    # folder          # template alias
    'storage/static': 'static/',
    ...
    'storage/public': '/'
}
```

{% endcode %}

You will now be able to access `localhost:8000/robots.txt` and you will have your robots.txt served correctly and it can be indexed by search engines properly.

Thats it! Static files are extremely simple. You are now a master at static files!
