# Encryption

## Introduction

Masonite comes with bcrypt out of the box but leaves it up to the developer to actually encrypt things like passwords. You can opt to use any other hashing library but bcrypt is the standard of a lot of libraries and comes with some one way hashing algorithms with no known vulnerabilities. Many of hashing algorithms like SHA-1 and MD5 are not secure and you should not use them in your application. 

{% hint style="success" %}
You can read the [bcrypt documentation here](https://github.com/pyca/bcrypt).
{% endhint %}

## Background

Also, we make sure that Javascript cannot read your cookies. It's important to know that although your website may be secure, you are susceptible to attacks if you import third party Javascript packages \(since those libraries could be hackable\) which can read all cookies on your website and send them to the hacker.

Other frameworks use cryptographic signing which attached a special key to your cookies that prevents manipulation. This does't make sense as a major part of XSS protection is preventing third parties from reading cookies. It doesn't make sense to attach a digital signature to a plaintext cookie if you don't want third parties to see the cookie \(such as a session id\). Masonite takes one step further and encrypts the entire string and can only be decrypted using your secret key \(so make sure you keep it secret\).

## Secret Key

In your `.env` file, you will find a setting called `KEY=your-secret-key`. This is the SALT that is used to encrypt and decrypt your cookies. It is important to change this key sometime before development. You can generate new secret keys by running:

```text
$ craft key
```

This will generate a new key in your terminal which you can copy and paste into your `.env` file. Your `config/application.py` file uses this environment variable to set the `KEY` configuration setting.

Additionally you can pass the `--store` flag which will automatically set the `KEY=` value in your `.env` file for you:

```text
$ craft key --store
```

{% hint style="danger" %}
Remember to not share this secret key as a loss of this key could lead to someone being able to decrypt any cookies set by your application. If you find that your secret key is compromised, just generate a new key.
{% endhint %}

## Cryptographic Signing

You can use the same cryptographic signing that Masonite uses to encrypt cookies on any data you want. Just import the `masonite.sign.Sign` class. A complete signing will look something like:

```python
from masonite.auth.Sign import Sign

sign = Sign()

sign.encrypt('value') # PSJDUudbs87SB....

sign.decrypt('value') # 'value'
```

By default, `Sign()` uses the encryption key in your `config/application.py` file but you could also pass in your own key.

```python
from masonite.auth.Sign import Sign

encryption_key = b'SJS(839dhs...'

sign = Sign(encryption_key)

sign.encrypt('value') # PSJDUudbs87SB....

sign.decrypt('value') # 'value'
```

This feature uses [pyca/cryptography](https://cryptography.io/en/latest/) for this kind of encryption. Because of this, we can generate keys using Fernet.

```python
from masonite.auth.Sign import Sign
from cryptography.fernet import Fernet

encryption_key = Fernet.generate_key()

sign = Sign(encryption_key)

sign.encrypt('value') # PSJDUudbs87SB....

sign.decrypt('value') # 'value'
```

Just remember to store the key you generated or you will not be able to decrypt any values that you encrypted.

## Using bcrypt

Bcrypt is very easy to use an basically consists of a 1 way hash, and then a check to verify if that 1 way hash matches an input given to it. 

{% hint style="warning" %}
It's important to note that any values passed to bcrypt need to be in bytes.
{% endhint %}

### Hashing Passwords

Again, all values passed into bcrypt need to be in bytes so we can pass in a password:

```python
password = bcrypt.hashpw(
                bytes(request.input('password'), 'utf-8'), bcrypt.gensalt()
            )
```

Notice that the value passed in from the request was converted into bytes using the `bytes()` Python function.

Once the password is hashed, we can just safely store it into our database

```python
User.create(
    name=request.input('name'),
    password=password,
    email=request.input('email'),
)
```

{% hint style="danger" %}
Do not store unhashed passwords in your database. Also, do not use unsafe encryption methods like MD5 or SHA-1.
{% endhint %}

### Checking Hashed Passwords

In order to check if a password matches it's hashed form, such as trying to login a user, we can use the `bcrypt.checkpw()` function:

```python
bcrypt.checkpw(bytes('password', 'utf-8'), bytes(model.password, 'utf-8'))
```

This will return true if the string `'password'` is equal to the models password.

More information on bcrypt can be found by reading it's documentation.

