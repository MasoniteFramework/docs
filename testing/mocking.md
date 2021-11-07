When it comes to unit testing, you always want to test a unit of your piece of code. Your code might
depends on third party services such as an API and you don't want to call it during your local tests
or in your CI environment. That's when you should use mocking to mock the external parts or the part
you don't want to test.

Masonite comes with some mocking abilities for some of the features relying on third party services.
For other parts or you own code you can use Python mocking abilities provided by `unittest.mock` standard module.

## Masonite Features Mocks

Masonite tests case have two helpers method `fake()` and `restore()`.

You can mock a Masonite feature by doing `self.fake(feature)` and then restore it to the real feature behaviour
by calling `self.restore(feature)`. When a feature is mocked the real behaviour won't be called, instead
a quick and simple implementation is ran, often offering the ability to inspect and test what happens.

Available features that can be mocked (for now) are:

- [Mail](/features/mail)
- [Notification](/features/notifications)

### Mocking Mail

When mocking emails it will prevent emails from being really sent. Typically, sending mail is unrelated to the code you are actually testing. Most likely, it is sufficient to simply assert that Masonite was instructed to send a given mailable.

Here is an example of how to mock emails sending in your tests:

```python
def setUp(self):
    super().setUp()
    self.fake("mail")

def tearDown(self):
    super().tearDown()
    self.restore("mail")

def test_mock_mail(self):
    welcome_email = self.application.make("mail").mailable(Welcome()).send()
    (
        welcome_email.seeEmailContains("Hello from Masonite!")
        .seeEmailFrom("joe@masoniteproject.com")
        .seeEmailCountEquals(1)
    )
```

Available assertions are:

- seeEmailWasSent()
- seeEmailWasNotSent()
- seeEmailCountEquals(count)
- seeEmailTo(string)
- seeEmailFrom(string)
- seeEmailReplyTo(string)
- seeEmailBcc(string)
- seeEmailCc(string)
- seeEmailSubjectEquals(string)
- seeEmailSubjectContains(string)
- seeEmailSubjectDoesNotContain(string)
- seeEmailContains(string)
- seeEmailDoesNotContain(string)
- seeEmailPriority(string)

### Mocking Notification

When mocking notifications it will prevent notifications from being really sent. Typically, sending notification is unrelated to the code you are actually testing. Most likely, it is sufficient to simply assert that Masonite was instructed to send a given notification.

Here is an example of how to mock notifications sending in your tests:

```python
def setUp(self):
    super().setUp()
    self.fake("notifications")

def tearDown(self):
    super().tearDown()
    self.restore("notifications")

def test_mock_notification(self):
    notification = self.application.make("notification")
    notification.assertNothingSent()
    notification.route("mail", "test@mail.com").send(WelcomeNotification())
    notification.route("mail", "test@mail.com").send(WelcomeNotification())
    notification.assertCount(2)
```

```python
def test_mock_notification(self):
    self.application.make("notification").route("mail", "test@mail.com").route(
        "slack", "#general"
    ).send(OrderNotification(10))
    self.application.make("notification").assertLast(
        lambda user, notif: (
            notif.assertSentVia("mail")
            .assertEqual(notif.order_id, 10)
            .assertEqual(
                notif.to_slack(user).get_options().get("text"),
                "Order 10 has been shipped !",
            )
        )
    )
```

Available assertions are:

- assertNothingSent()
- assertCount(count)
- assertSentTo(notifiable, notification_class, callable_assert=None, count=None)
- assertLast(callable_assert)
- assertNotSentTo(notifiable, notification_class)

On Notifications instances:

- assertSentVia(\*drivers)
- assertEqual(value, reference)
- assertNotEqual(value, reference)
- assertIn(value, container)

Available helpers are:

- resetCount()
- last()

## Basic Python mocks

For mocking any piece of code in Python you can use the standard `unittest.mock` module. You can find
more information in [unittest documentation](https://docs.python.org/3/library/unittest.mock.html).

Here is basic example

```python
from unittest.mock import patch

with patch("some.module") as SomeClass:
    SomeClass.return_value.my_method.return_value = 0
    self.assertEqual(SomeClass().my_method(), 0)
```

For mocking external HTTP requests you can use the `responses` module. You can find more information
in [responses documentation](https://github.com/getsentry/responses).

```python
import responses

@responses.activate
def test_mock_third_party_api(self):
    responses.add(responses.POST, "api.github.com", body=b"Ok")
    # do somehting in your code
    self.assertTrue(responses.assert_call_count("api.github.com", 1))
```
