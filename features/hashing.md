Masonite provides secure hashing for storing user passwords or other data. Bcrypt and Argon2 protocols
can be used with Masonite (default is Bcrypt).

# Configuration

Hashing configuration is located at `config/application.py` file. In this file, you can configure which protocol
to use.

{% code title="config/application.py" %}
```python
HASHING = {
    "default": "bcrypt",
    "bcrypt": {"rounds": 10},
    "argon2": {"memory": 1024, "threads": 2, "time": 2},
}
```
{% endcode %}

# Hashing a string

You can use the `Hash` facade to easily hash a string (e.g. a password):

```python
from masonite.facades import Hash

Hash.make("secret") #== $2b$10$3Nm9sWFYhi.GUJ...
```

Note that you can return a hash as bytes with:

```python
from masonite.facades import Hash

Hash.make_bytes("secret") #== b"$2b$10$3Nm9sWFYhi.GUJ..."
```

# Checking a string matches a Hash

To check that a plain-text string corresponds to a given hash you can do:

```python
from masonite.facades import Hash

Hash.check("secret", "$2b$10$3Nm9sWFYhi.GUJ...") #== True
```

# Verifying a Hash needs to be re-hashed

You can determine if the work factor used by the hashing protocol has changed since the string was hashed using `needs_rehash`:

```python
from masonite.facades import Hash

Hash.needs_rehash("$2b$10$3Nm9sWFYhi.GUJ...") #== True
```

# Options

You can change hashing protocol configuration on the fly for all Hash methods:

```python
from masonite.facades import Hash

Hash.make("secret", options={"rounds": 5})
```

You can also change protocol on the fly:

```python
from masonite.facades import Hash

Hash.make("secret", driver="argon2", options={"memory": 512, "threads": 8, "time": 2})
```
