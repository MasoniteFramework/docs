# Masonite AuthHub

# Introduction

Masonite AuthHub brings a centralized and easy to integrate OAuth system to the Masonite Framework. Simply add a few lines of code and the entire OAuth workflow is done for you.

### Getting Started

To install Masonite AuthHub just pip install it:

```
$ pip install authhub
```

After `authhub` is installed, we just need to publish it.

#### Adding The Configuration File

Masonite AuthHub uses the `config/services.py` configuration file. Conveniently, AuthHub comes with a `publish` command we can use to create this.

#### NOTE: Virtual Environment

**If you are in a virtual environment, **`craft publish`** will not have access to your virtual environment dependencies. In order to fix this, we can add our site packages to our **`config/packages.py`** config file**

If you are in a virtual environment then go to your `config/packages.py` file and add your virtual environments site\_packages folder to the `SITE_PACKAGES` list. Your `SITE_PACKAGES` list may look something like:

```python
SITE_PACKAGES = [
    'venv/lib/python3.6/site-packages'
]
```

This will allow `craft publish` to find our dependencies installed on our virtual environment. Read the [Publishing Packages](/publishing-packages.md) documentation for more information.

### Publishing

Publish AuthHub by running:

```
$ craft publish authhub
```

This will create or append to the `config/services.py` file. If you've published a package that has used the `config/services.py` file before than you may have to take the contents of the `AUTH_PROVIDERS` dictionary that was created and condense it down into a single dictionary.

After we have published AuthHub we should get a dictionary that looks like:

```python
AUTH_PROVIDERS = {
    'github': {
        'client': os.environ.get('GITHUB_CLIENT'),
        'secret': os.environ.get('GITHUB_SECRET'),
        'redirect': os.environ.get('GITHUB_REDIRECT'),
        'driver': 'authhub.providers.GitHubDriver.GitHubDriver'
    }
}
```

Just add the corresponding environment variables to your `.env` file:

```
GITHUB_CLIENT=XXX
GITHUB_SECRET=XXX
GITHUB_REDIRECT=http://your-redirect-url/
```

The `GITHUB_REDIRECT` url is the url that users will return to after they authenticate. This is likely to match the return URL in your app on the provider you are using. For GitHub, this is called “Authorization callback URL” in your OAuth App’s settings.

### Redirecting

AuthHub uses the same syntax for all providers and contains a method of redirecting to the provider as well as a method of getting the response.

To redirect to the provider so you can authorize the user:

```python
return AuthHub(request).driver('github').redirect()
```

Notice the `.driver()` method here. This driver will instantiate the driver specified in your configuration setting in the previous step.

If you need to, you can also specify some scopes:

```python
return AuthHub(request).driver('github').scope('repo', 'public_repo').redirect()
```

Or pass in a state:

```python
return AuthHub(request).driver('github').scope('repo', 'public_repo').state('secret_id').redirect()
```

The state will be a value returned back after the user has authenticated. This is good for verifying if the user that sent the request is the one that received it.

To get the user response back after the user has authenticated:

```python
return AuthHub(request).driver('github').user()
```

What this method does is:

* receives the response back from the provider
* gets the code from the query string
* exchanges the code for an access token
* uses the access token to retrieve and return the user.

Pretty cool, huh?

A complete setup might look something like:

```python
from authhub.authhub import AuthHub

class LoginController(object):
    ''' Class Docstring Description '''

    def __init__(self):
        pass

    def toProvider(self, request):
        return AuthHub(request).driver('github').redirect()

    def fromProvider(self, request):
        user = AuthHub(request).driver('github').user()
        return user['login'] # returns github username
```

Thats it! Check your platform’s typically response in order to see what is in the  user object. It’s a good idea to store the access token in your `app/User` table and use that token to perform API requests on behest of the user. Many providers like GitHub, Facebook and Twitter all have great Python libraries you can use the token with.