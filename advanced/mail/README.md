# Mail

## Email

## Introduction

Masonite comes with email support out of the box. Most applications will need to send email upon actions like account creation or notifications. Because email is used so often with software applications, masonite provides mail support with several drivers.

### Getting Started

All mail configuration is inside `config/mail.py` and contains several well documented options. There are several built in drivers you can use but you can make your own if you'd like. You can follow the documentation here at "Making an Email Driver." If you do make your own, consider making it available on PyPi so others can install it. We may even put it in Masonite by default.

By default, Masonite uses the `smtp` driver. Inside your `.env` file, just put your smtp credentials. If you are using Mailgun then switch your driver to `mailgun` and put your Mailgun credentials in your `.env` file.

### Sending an Email

The `Mail` class is loaded into the container via the the `MailProvider` Service Provider. We can fetch this `Mail` class via our controller methods like:

```python
def show(self, Mail):
    print(Mail) # returns Mail class
```

We can send an email like so:

```python
def show(self, Mail):
    Mail.to('hello@email.com').send('Welcome!')
```

That's it! There are a few methods we can attach to extend how we send email.

### Methods

We can specify which driver we want to use. Although Masonite will use the `DRIVER` variable in our config file, we can change the driver on the fly:

```text
Mail.driver('mailgun').to('hello@email.com').send('Welcome!')
```

We can also specify the subject:

```text
Mail.subject('Welcome!').to('hello@email.com').send('Welcome!')
```

You can specify which address you want the email to appear from:

```text
Mail.send_from('Admin@email.com').to('hello@email.com').send('Welcome!')
```

#### Templates

If you don't want to pass a string as the message, you can pass a view template.

```text
Mail.to('idmann509@gmail.com').template('mail/welcome').send()
```

This will render the view into a message body and send the email as html. Notice that we didn't pass anything into the `send` message

