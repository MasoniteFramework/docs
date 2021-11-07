Masonite also provides a simple way to authorize user actions against a given resource. This is achieved with two concepts: gates and policies. Gates are as the name suggests an authorization check that you will be able to invoke to verify user access. Policies are a way to groupe authorization logic around a model.

# Gates

Gates are simple callable that will define if a user is authorized to perform a given action.
A handy `Gate` facade is available to easily manipulate gates.

## Registering Gates

Gates receive a user instance as their first argument and may receive additionals arguments such as
a Model instance. You needs to define `Gates` in `boot()` method of your service provider.

In the following example we are adding gates to verify if a user can create and update posts. Users
can create posts if they are administrators and can update posts they have created only.

```python
from masonite.facades import Gate

class MyAppProvider(Provider):

    def boot(self):
        Gate.define("create-post", lambda user: user.is_admin)
        Gate.define("update-post", lambda user, post: post.user_id == user.id)
```

You can then check if a gate exists or has been registed by using `has()` which is returning
a boolean:

```python
Gate.has("create-post")
```

{% hint style="info" %}
If a unknown gate is used a `GateDoesNotExist` exception will be raised.
{% endhint %}

## Authorizing Actions

Then anywhere (often in a controller) in your code you can use those gates to check if the **current authenticated user** is authorized to perform the given action defined by the gate.

Gates exposes different methods to perform verification: `allows()`, `denies()`, `none()`, `any()`, `authorize()` `inspect()`.

`allows()`, `denies()`, `none()` and `any()` return a boolean indicating if user is authorized

```python
if not Gate.allows("create-post"):
    return response.redirect("/")

# ...
post = Post.find(request.input("id"))
if Gate.denies("update-post", post):
    return response.redirect("/")

# ...
Gate.any(["delete-post", "update-post"], post)

# ...
Gate.none(["force-delete-post", "restore-post"], post)
```

`authorize()` does not return a boolean but will raise an `AuthorizationException` exception instead that will be rendered as an HTTP response with a 403 status code.

```python
Gate.authorize("update-post", post)
# if we reach this part user is authorized
# else an exception has been raised and be rendered as a 403 with content "Action not authorized".
```

Finally for better control over the authorization check you can analyse the response with `inspect()`:

```python
response = Gate.inspect("update-post", post)
if response.allowed():
     # do something
else:
     # not authorized and we can access message
     Session.flash("errors", response.message())
```

## Via the User Model

`Authorizes` class can be added to your User model to allow handy permissions checking:

```python
from masonite.authentication import Authenticates
from masonite.authorization import Authorizes

class User(Model, Authenticates, Authorizes):
```

A fluent authorization api is now available on user instances:

```python
user.can("delete-post", post)
user.cannot("access-admin")
user.can_any(["delete-post", "force-delete-post"], post)
```

All of those methods receive the gate name as first argument and then some additional arguments if required.

## With a given user

You can use the `for_user()` method on the Gate facade to make the verification against a given user instead
of the authenticated user.

```python
user = User.find(1)
Gate.for_user(user).allows("create-post")
```

## Gates Hooks

During gate authorization process `before` and `after` hooks can be triggered.

A `before` hook can be added like this :

```python
# here admin users will always be authorized whatever is happening after
Gate.before(lambda user, permission : user.role == "admin")
```

The `after` hook is working the same way excepted that it will receive the authorization result.

```python
Gate.after(lambda user, permission, result : user.role == "admin")
```

If after callback is returning a value it will take precedance over the gate result check.

# Policies

Policies are classes that organize authorization logic around a particular model.

## Creating Policies

You can create a policy by calling

```
$ python craft policy AccessAdmin
```

You can also create a model policy (with a set of predefined gates) by calling

```
$ python craft policy Post --model
```

A model policy comes with the common actions that we can perform on a Model:

```python
from masonite.authorization import Policy


class PostPolicy(Policy):
    def create(self, user):
        return False

    def view_any(self, user):
        return False

    def view(self, user, instance):
        return False

    def update(self, user, instance):
        return False

    def delete(self, user, instance):
        return False

    def force_delete(self, user, instance):
        return False

    def restore(self, user, instance):
        return False
```

You are free to add any other methods on your policies:

```python
from masonite.authorization import Policy


class PostPolicy(Policy):
    def publish(self, user):
        return user.email == "admin@masonite.com"

```

## Registering Policies

Then in your service provider (as for defining gates) you should register the policies and bind them with a model:

```python
from masonite.facades import Gate

class MyAppProvider(Provider):

    def boot(self):
        Gate.register_policies(
            [Post, PostPolicy],
            [User, UserPolicy]
        )
```

A policy for the Post model can looke like this:

```python
from masonite.authorization import Policy

class PostPolicy(Policy):
    def create(self, user):
        return user.email == "idmann509@gmail.com"

    def view(self, user, instance):
        return True

    def update(self, user, instance):
        return user.id == instance.user_id
```

{% hint style="info" %}
If a unknown policy is used a `PolicyDoesNotExist` exception will be raised.
{% endhint %}

## Authorizing Actions

You can then use the `Gate` facade methods to authorize actions defined in your policies (as for the gates). With the previously defined `PostPolicy` we could for example make the following verifications:

```python
post = Post.find(1)
Gate.allows("update", post)
Gate.denies("view", post)
Gate.allows("force_delete", post)
```

The `create()` or `view_any()` methods do not take a model instance, that is why the model class should be
provided so that Gate mechanism can infer from which policy those methods are belonging to.

```python
Gate.allows("create", Post)
Gate.allows("view_any", Post)
```
