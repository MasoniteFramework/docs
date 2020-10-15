# Seeding

Seeding is simply a way to quickly seed, or put data into your tables.

## Creating Seeds

You can create a seed file and seed class which can be used for keeping seed information and running it later.

To create a seed run the command:

```text
$ masonite-orm seed User
```

This will create some boiler plate for your seeds that look like this:

```python
from masonite.orm.seeds import Seeder

class UserTableSeeder(Seeder):

    def run(self):
        """Run the database seeds."""
        pass
```

From here you can start building your seed.

## Building Your Seed

A simple seed might be creating a specific user that you use during testing.

```python
from masonite.orm.seeds import Seeder
from models import User

class UserTableSeeder(Seeder):

    def run(self):
        """Run the database seeds."""
        User.create({
            "username": "Joe",
            "email": "joe@masoniteproject.com",
            "password": "secret"
        })
```

## Running Seeds

You can easily run your seeds:

```text
$ masonite-orm seed:run User
```

## Database Seeder

## Factories

Factories are simple and easy ways to generate mass amounts of data quickly. You can put all your factories into a single file.

### Creating A Factory Method

Factory methods are simple methods that take a single `Faker` instance.

```python
# config/factories.py

def user_factory(self, faker):
    return {
        'name': faker.name(),
        'email': faker.email(),
        'password': 'secret'
    }
```

For methods available on the `faker` variable reference the [Faker](https://faker.readthedocs.io/en/master/) documentation.

### Registering Factories

Once created you can register the method with the `Factory` class:

```python
# config/factories.py
from masonite.orm import Factory
from models import User

def user_factory(self, faker):
    return {
        'name': faker.name(),
        'email': faker.email(),
        'password': 'secret'
    }

Factory.register(User, user_factory)
```

### Naming Factories

If you need to you can also name your factories so you can use different factories for different use cases:

```python
# config/factories.py
from masonite.orm import Factory
from models import User

def user_factory(self, faker):
    return {
        'name': faker.name(),
        'email': faker.email(),
        'password': 'secret'
    }

def admin_user_factory(self, faker):
    return {
        'name': faker.name(),
        'email': faker.email(),
        'password': 'secret',
        'is_admin': 1
    }

Factory.register(User, user_factory)
Factory.register(User, admin_user_factory, name="admin_users")
```

### Calling Factories

To use the factories you can import the `Factory` class from where you built your factories. In our case it was the `config/factories.py` file:

```python
from config.factories import Factory
from models import User

users = Factory(User, 50).create() #== <masonite.orm.collections.Collection object>
user = Factory(User).create() #== <models.User object>
```

This will persist these users to the database. If you want to simply make the models or collection \(and not persist them\) then use the `make` method:

```python
from config.factories import Factory
from models import User

users = Factory(User, 50).make() #== <masonite.orm.collections.Collection object>
user = Factory(User).make() #== <models.User object>
```

Again this will NOT persist values to the database.

### Calling Named Factories

By default, Masonite will use the factory you created without a name. If you named the factories you can call those specific factories easily:

```python
from config.factories import Factory
from models import User

users = Factory(User, 50).create(name="admin_users") #== <masonite.orm.collections.Collection object>
```

### Modifying Factory Values

If you want to modify any values you previously set in the factory you created, you can pass a dictionary into the `create` or `make` method:

```python
from config.factories import Factory
from models import User

users = Factory(User, 50).create({'email': 'john@masoniteproject.com'}) #== <masonite.orm.collections.Collection object>
```

This is a great way to make constant values when testing that you can later assert to.

