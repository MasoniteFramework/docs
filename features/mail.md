Masonite has a simple yet powerful mail feature which is used to send emails from your application. 

# Creating Emails

To create and send an email with Masonite, you must first built a `Mailable` class. This class will act as the settings for your email such as the address you are sending from, the subject, the text of the email, the html of the email, etc.

All of these settings on the mailable can be changed or overwritten when you are ready to send you email, more on this later on.

The first step of building a mailable is running the command:

```terminal
$ python craft mailable Welcome
```

This will create your mailable and it will look something like this:

```python
class Welcome(Mailable):
    def build(self):
        (
            self.subject("Welcome to our site!")
            .from_("admin@example.com")
            .view("mailables.welcome", {})
        )
```

# Sending Mailables

You can send your mailable inside your controller easily by using the `Mail` class:

```python
from masonite.mail import Mail
from app.mailables.Welcome import Welcome

class WelcomeController(Controller):
  
    def welcome(self, mail: Mail):
        mail.mailable(Welcome().to('user@example.com')).send()
```

Notice at this point you can call any building options you want on the mailables to modify the behavior of it before sending.

# Mail Options

You can modify the behavior of the mailable by using any one of these options

| Options                                 | Description                                                  |
| --------------------------------------- | ------------------------------------------------------------ |
| `to('user@example.com')`                | Specifies the user to send the email to. <br />You may also specify the users name like `to('Joseph <user@example.com>')`. |
| `from_("admin@example.com")`            | Specifies the address that the email should appear it is from. |
| `cc(["accounting@example.com"])`        | A list of the addresses that should be "carbon copied" onto this email |
| `bcc(["accounting@example.com"])`       | A list of the addresses that should be "blind carbon copied" onto this email |
| `subject('Subject of the Email')`       | Specifies the subject of the email.                          |
| `reply_to('customers@example.com')`     | Specifies the address that will be set if a user clicks reply to this email |
| `text('Welcome to Masonite')`           | Specifies the text version of the email.                     |
| `html('Welcome to Masonite')`           | Specifies the HTML version of the email.                     |
| `view('mailables.view', {})`            | Specifies a view file with data to render the HTML version of the email |
| `priority(1)`                           | Specifies the priority of the email, values should be 1 through 5. |
| `low_priority()`                        | Sets the priortiy of the email to 1                          |
| `high_priority()`                       | Sets the priortiy of the email to 5                          |
| `attach('MAY.pdf', 'path/invoice.pdf')` | Attaches a file to the email                                 |

# Sending Attachments

Sending attachments is really simply with Masonite. Simply attach the file to the mailable before sending it:

```python
user = user.find(1)
welcome_mailable = WelcomeMailable().to(f"{user.name} <{user.email}>')
welcome_mailable.attach("MAY-2021-invoice.pdf", "storage/pdf/invoice.pdf")
mail.mailable(welcome_mailable).send()
```

You will then see your attachment in the email.

# Mailable Responses

When you are building your emails it might be nice to see how they look before sending them. This can save a lot of time when you're trying to get those styles just right.

You can simply return your mailable in your controller and it will return like a normal view file.

```python
from app.mailables.Welcome import Welcome

class WelcomeController(Controller):
  
    def welcome(self):
        return Welcome()
```

If you are using the `view()` option in your mailable then you will need to set the application on the mailable:

```python
from app.mailables.Welcome import Welcome
from wsgi import application

class WelcomeController(Controller):
  
    def welcome(self):
        return Welcome().set_application(application)
```

# Changing Drivers

You can change the driver which sends the email by using the `driver` argument in the `send()` method:

```python
mail.send(Welcome().to('user@example.com'), driver="smtp")
```

