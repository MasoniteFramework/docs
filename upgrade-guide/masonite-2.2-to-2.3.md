# Preface

Welcome to Masonite 2.3! In this guide we will walk you through how to upgrade your Masonite application from version 2.2 to version 2.3.

In this guide we will only be discussing the breaking changes and won't talk about how to refactor your application to use the awesome new features that 2.3 provides. For that information you can check the Whats New in 2.3 documentation to checkout the new features and how to refactor.

We'll walk through both Masonite upgrades and breaking packages as well

# Masonite

## Upgrade Your Masonite and CLI Version

Let's first start by upgrading your Masonite version. Depending on your dependancy manager it will look something like this:

Change it from this:

```
masonite>=2.2,<2.3
```

to 

```
masonite>=2.3,<2.4
```

Now let's make sure our cli tool is up to date as well:

```
$ pip install masonite-cli --upgrade
```

## Change Server Response

Masonite changed the way the response is generated internally so you will need to modify how the response is retrieved internally. To do this you can go to your `bootstrap/start.py` file and scroll down to the bottom.

Change it from this:

```python
return iter([bytes(container.make('Response'), 'utf-8')])
```

to this:

```python
return iter([container.make('Response')])
```

This will allow Masonite to better handle responses. Instead of converting everything to a string like the first snippet we can now return bytes. This is useful for returning images and documents.

## Package Requirements

Previously Masonite used several packages and required them by default to make setting everything up easier. This slows down package development because now any breaking upgrades for a package like Masonite Validation requires waiting for the the next major upgrade to make new breaking features and improve the package.

Now Masonite no longer requires these packages by default and requires you as the developer to handle the versioning of them. This allows for more rapid development of some of Masonite packages.

Masonite packages also now use SEMVER versioning. This is in the format of `MAJOR.MINOR.PATCH`. Here are the required versions you will need for Masonite 2.3:

* masonite-validation>=3.0.0
* masonite-scheduler>=3.0.0

These are the only packages that came with Masonite so you will need to now manage the dependencies on your own. It's much better this way.

## Authentication has been completely refactored

Masonite now uses a concept called guards so you will need a quick crash course on guards. Guards are simply logic related to logging in, registering, and retrieving users. For example we may have a `web` guard which handles users from a web perspective. So registering, logging in and getting a user from a database and browser cookies.

We may also have another guard like `api` which handles users via a JWT token or logs in users against the API itself.

Guards are not very hard to understand and are actually unnoticable unless you need them.

In order for the guards to work properly you need to change your `config/auth.py` file to use the newer configuration settings.

You'll need to change your settings from this:

```python
AUTH={
    'driver': env('AUTH_DRIVER', 'cookie'),
    'model': User,
}
```

to this:

```python
AUTH = {
    'defaults': {
        'guard': 'web'
    },
    'guards': {
        'web': {
            'driver': 'cookie',
            'model': User,
            'drivers': { # 'cookie', 'jwt'
                'jwt': {
                    'reauthentication': True,
                    'lifetime': '5 minutes'
                }
            }
        },
    }
}
```

## Add The Authentication Provider

To manage the guards (and register new guards) there is the new `AuthenticationProvider` that needs to be added to your providers list.

```python
from masonite.providers import AuthenticationProvider

PROVIDERS = [
    # Framework Providers
    # ...
    AuthenticationProvider,

    # Third Party Providers
    #...
]
```

## Removed The Sass Provider

Masonite no longer supports SASS and LESS compiling. Masonite now uses webpack and NPM to compile assets. You will need to now reference the Compiling Assets documentation.

You will need to remove the `SassProvider` completely from the providers list in `config/providers.py`. As well as remove the `SassProvider` import from on top of the file.

You can also completely remove the configuration settings in your `config/storage.py` file:

```python
SASSFILES = {
    'importFrom': [
        'storage/static'
    ],
    'includePaths': [
        'storage/static/sass'
    ],
    'compileTo': 'storage/compiled'
}
```

Be sure to reference the Compiling Assets documentation to know how to use the new NPM features.

## Removed The Ability For The Container To Hold Modules

The container can no longer hold modules. Modules now have to be imported in the class you require them. For example you can't bind a module like this:

```
from config import auth

app.bind('AuthConfig', auth)
```

and then make it somewhere else:

```python
class ClassA:

    def __init__(self, container: Container):
        self.auth = container.make('AuthConfig')
```

This will throw a `StrictContainerError` error. Now you have to import it so will have to do something like this using the example above:

```python
from config import auth

class ClassA:

    def __init__(self, container: Container):
        self.auth = auth
```

## Remove Modules from Container

Now that we can no longer bind modules to the container we need to make some changes to the `wsgi.py` file because we did that here.

Around line 16 you will see this:

```python
container.bind('Application', application)
```

Just completely remove that line. Its no longer needed.

Also around line 19 you will see this line:

```python
container.bind('ProvidersConfig', providers)
```

You can completely remove that as well.

Lastly, around line 31 you can change this line:

```python
for provider in container.make('ProvidersConfig').PROVIDERS:
```

to this:

```python
for provider in providers.make('ProvidersConfig').PROVIDERS:
```

## Changed How Query Strings Are Parsed

It's unlikely this effects you and query string parsing didn't change much but if you relied on query strings like this:

`/?filter[name]=Joe&filter[user]=bob&email=user@email.com`

Or html elements like this:

```
<input name="options[name]" value="Joe">
<input name="options[user]" value="bob">
```

then query strings will now parse to:

```python
{
    "email": "user@email.com",
    "options": {
        "name": "Joe",
        "user": "bob"
    }
}
```

You'll have to update any code that uses this. If you are not using this then don't worry you can ignore it. 

# Scheduler Namespace 

Not many breaking changes were done to the scheduler but there are alot of new features. Head over to the Whats New in Masonite 2.3 section to read more.

We did change the namespace from `scheduler` to `masonite.scheduler`. So you will need to refactor your imports if you are using the scheduler.

# Pip uninstall masonite-cli

Craft is now a part of Masonite core so you can uninstall the masonite-cli tool. You now no longer need to use that as a package.

```
$ pip uninstall masonite-cli
```

# Conclusion

You should be all good now! Try running your tests or running `craft serve` and browsing your site and see if there are any issues you can find. If you ran into a problem during upgrading that is not found in this guide then be sure to reach out so we can get the guide upgraded.
