Masonite testing is very simple. You can test very complex parts of your code with ease by just extending your class with a Masonite unit test class.

Although Masonite uses `pytest` to run tests, Masonite's test suite is based on `unittest`. So you will use `unittest` syntax but run the tests with `pytest`. Because of this, all syntax will be in camelCase instead of PEP 8 lower_case_with_underscores. Just know that all TestCase assertions used during testing is in camelCase form to maintain unittest standards.

## Getting Started

### Environment

When running tests, Masonite will automatically set the environment to `testing`. You are free to define other testing environment configuration values as necessary.

You can create a `.env.testing` file. Feel free to load any testing environment variables in here. By default they will not be commited. When `pytest` runs it will additionally load and override any additional environment variables.

### Creating tests

You can simply create a file starting with `test_` and then creating a test class inheriting from masonite `TestCase` class.

You can also directly use the command

```python
$ python craft test SomeFeatureTest
```

to create `tests/unit/test_some_feature.py`:

```python
from masonite.tests import TestCase


class SomeFeatureTest(TestCase):
    def setUp(self):
        super().setUp()

    def test_something(self):
        self.assertTrue(True)
```

That's it! You're ready to start testing. Read on to learn how to build your test cases and run them.

### Running tests

You can run tests by calling

```
$ python -m pytest
```

This will automatically discover your tests following pytest [automatic tests discovery](https://docs.pytest.org/en/6.2.x/goodpractices.html#test-discovery).
You can also run a specific test class

```
$ python -m pytest tests/unit/test_my_feature.py
```

Or a specific test method

```
$ python -m pytest tests/unit/test_my_feature.py::MyFeatureTest::test_feature_is_working
```

Finally you can re-run the last failed tests automatically

```
$ python -m pytest --last-failed
```

## Building test cases

### Test Life Cycle

When you run a test class each test method of this test class will be ran following a specific
life cycle.

```python
class TestFeatures(TestCase):

    @classmethod
    def setUpClass(cls):
        """Called once before all tests of this class are executed."""
        print("Setting up test class")

    @classmethod
    def tearDownClass(cls):
        """Called once after all tests of this class are executed."""
        print("Cleaning up test class")

    def setUp(self):
        """Called once before each test are executed."""
        super().setUp()
        print("Setting up individual unit test")

    def tearDown(self):
        """Called once after each test are executed."""
        super().tearDown()
        print("Cleaning up individual unit test")

    def test_1(self):
        print("Running test 1")

    def test_2(self):
        print("Running test 2")
```

Running the above test class will create this output:

```
Setting up test class
Setting up individual unit test
Running test 2
Cleaning up individual unit test
Setting up individual unit test
Running test 1
Cleaning up individual unit test
Cleaning up test class
```

{% hint style="warning" %}
Note that tests methods are not always ran in the order specified in the class. Anyway you should
not make the assumptions that tests will be run in a given order. You should try to make your
tests idempotent.
{% endhint %}

### Chaining assertions

All methods that begin with `assert` can be chained together to run through many assertions. All other method will return some kind of boolean or value which you can use to do your own assertions.

### Asserting exceptions

Sometimes you need to assert that a given piece of code will raise a given exception. To do this you
can use the standard `assertRaises()` context manager:

```python
with self.assertRaises(ValidationError) as e:
    # run some code here
    raise ValidationError("An error occured !")

self.assertEqual(str(e.exception), "An error occured !")
```

### Capturing test output

Sometimes you need to test the output of a function that prints to the console. To do this in your
tests you can use the `captureOutput()` context manager:

```python
with self.captureOutput() as output:
    # run some code here
    print("Hello World !")

self.assertEqual(output.getvalue().strip(), "Hello World !")
```

## HTTP tests

To make a request in your tests, you may use the `get`, `post`, `put`, `patch`, or `delete` methods within your test. These methods do not actually issue a "real" HTTP request to your application. Instead of returning a Masonit class `Response` instance, test request methods return a `HTTPTestResponse`instance, which provides a variety of helpful assertions that allow you to inspect and assert application's responses.

```python
def test_basic_request(self):
    self.get("/").assertOk()
```

### Getting request and response

The request and responses of a test can be fetch by accessing the `request` and `response` attributes.

```python
def test_request_and_response(self):
    request = self.get('/testing').request # <masonite.requests.Request>
    response = self.get('/testing').response # <masonite.response.Response>
```

### Registering routes

During tests you can register routes used only for testing purposes. For this you can use the
`addRoutes()` method at beginning of your tests:

```python
def setUpClass(cls):
    super().setUpClass()
    cls.addRoutes(
        Route.get("/", "TestController@show"),
        Route.post("/", "TestController@show"),
    )
```

### Checking if a route exists

To check if a route exists, we can simple use either `get` or `post`:

```python
def test_route_exists(self):
    self.assertTrue(self.get('/testing'))
    self.assertTrue(self.post('/testing'))
```

### Request Headers

You may use the `withHeaders()` method to customize the request's headers before it is sent to the application. This method allows you to add any headers you would like to the request by providing them as a dict:

```python
request = self.withHeaders({"X-TEST": "value"}).get("/").request
self.assertEqual(request.header("X-Test"), "value")
```

### Cookies

You may use the `withCookies()` method to set cookie values before making a request. This method accepts a dictionary of name / value pairs:

```python
self.withCookies({"test": "value"}).get("/").assertCookie("test", "value")
```

### Authentication

If you want to make authenticated requests in your tests, you can use the `actingAs()` method that
takes a given `User` record and authenticate him during the request.

```python
user = User.find(1)
self.actingAs(user).get("/")
```

### CSRF protection

By default, all calls to your routes with the above methods will be without CSRF protection. The testing code will allow you to bypass that protection.

This is very useful since you don't need to worry about setting CSRF tokens on every request but you may want to enable this protection. You can do so by calling the `withCsrf()` method on your test.

```python
def test_csrf(self):
    self.withCsrf()
    self.post("/unit/test/json", {"test": "testing"})
```

This will enable it on a specific test but you may want to enable it on all your tests. You can do this by adding the method to your `setUp()` method:

```python
def setUp(self):
    super().setUp()
    self.withCsrf()
```

Again you can disable this behaviour with `withoutCsrf()` method.

### Exceptions handling

As you have noticed, Masonite has exception handling which it uses to display useful information during development.

This is an issue during testing because we wan't to be able to see more useful testing related issues. Because of this, testing will disable Masonite's default exceptions handling and you will see more useful exceptions during testing. If you want to use Masonite's built in exceptions handling then you can enable it by running:

```python
def setUp(self):
    super().setUp()
    self.withExceptionHandling()
```

You can also disable exceptions handling again by using:

```python
def test_something(self):
    self.withExceptionHandling()
    self.get("/").assertUnauthorized()
    self.withoutExceptionHandling()
    self.get("/")
```

### Available Assertions

Masonite provides a variety of assertions methods to inspect and verify request/response logic when testing your application. Those assertions are available on the `HTTPTestResponse` returned by `get`, `post`, `put`, `patch`, or `delete`.

- [assertContains](#assertContains)
- [assertNotContains](#assertNotContains)
- [assertContainsInOrder](#assertContainsInOrder)
- [assertNoContent](#assertNoContent)
- [assertIsNamed](#assertIsNamed)
- [assertIsNotNamed](#assertIsNotNamed)
- [assertIsStatus](#assertIsStatus)
- [assertNotFound](#assertNotFound)
- [assertOk](#assertOk)
- [assertCreated](#assertCreated)
- [assertSuccessful](#assertSuccessful)
- [assertUnauthorized](#assertUnauthorized)
- [assertHasHeader](#assertHasHeader)
- [assertHeaderMissing](#assertHeaderMissing)
- [assertLocation](#assertLocation)
- [assertRedirect](#assertRedirect)
- [assertCookie](#assertCookie)
- [assertPlainCookie](#assertPlainCookie)
- [assertCookieExpired](#assertCookieExpired)
- [assertCookieNotExpired](#assertCookieNotExpired)
- [assertSessionHas](#assertSessionHas)
- [assertSessionMissing](#assertSessionMissing)
- [assertSessionHasErrors](#assertSessionHasErrors)
- [assertSessionHasNoErrors](#assertSessionHasNoErrors)
- [assertViewIs](#assertViewIs)
- [assertViewHas](#assertViewHas)
- [assertViewHasExact](#assertViewHasExact)
- [assertViewMissing](#assertViewMissing)
- [assertAuthenticated](#assertAuthenticated)
- [assertGuest](#assertGuest)
- [assertAuthenticatedAs](#assertAuthenticatedAs)
- [assertHasHttpMiddleware](#assertHasHttpMiddleware)
- [assertHasRouteMiddleware](#assertHasRouteMiddleware)
- [assertHasController](#assertHasController)
- [assertRouteHasParameter](#assertRouteHasParameter)
- [assertJson](#assertJson)
- [assertJsonPath](#assertJsonPath)
- [assertJsonExact](#assertJsonExact)
- [assertJsonCount](#assertJsonCount)
- [assertJsonMissing](#assertJsonMissing)

#### assertContains

Assert that returned response contains the given `content` string. Note that the response content will be eventually decoded if required.

```python
self.get("/").assertContains(content)
```

#### assertNotContains

Assert that returned response does not contain the given `content` string. Note that the response content will be eventually decoded if required.

```python
self.get("/").assertNotContains(content)
```

#### assertContainsInOrder

Assert that returned response contains in order the given strings. Note that the response content will be eventually decoded if required.

```python
self.get("/").assertContains(string1, string2, ...)
```

#### assertNoContent

Assert that returned response has no content and the given HTTP status code. The default status code that is asserted is 204.

```python
self.get("/").assertNoContent(status=204)
```

#### assertIsNamed

Assert that given route has the given name.

```python
self.get("/").assertIsNamed("home")
```

#### assertIsNotNamed

Assert that given route has not the given name.

```python
self.get("/").assertIsNotNamed("admin")
```

#### assertIsStatus

Assert that the response has the given HTTP status code:

```python
self.get("/").assertIsStatus(201)
```

#### assertOk

Assert that the response returns a 200 status code:

```python
self.get("/").assertOk()
```

#### assertCreated

Assert that the response returns a 201 status code:

```python
self.get("/").assertCreated()
```

#### assertSuccessful

Assert that the response has as status code between 200 and 300

```python
self.get("/").assertSuccessful()
```

#### assertUnauthorized

Assert that the response has as 401 status code

```python
self.get("/").assertUnauthorized()
```

#### assertForbidden

Assert that the response has as 403 status code

```python
self.get("/").assertForbidden()
```

#### assertHasHeader

Assert that the response has the given header name and value (if given).

```python
self.get("/").assertHasHeader(name, value=None)
```

#### assertHeaderMissing

Assert that the response does not have the given header.

```python
self.get("/").assertHeaderMissing(name)
```

#### assertLocation

Assert the response has the given URI value in `Location` header.

```python
self.get("/").assertLocation(uri)
```

#### assertRedirect

Assert that the response is a redirection to the given URI (if provided) or to the given route name with the given parameters (if provided).

```python
self.get("/logout").assertRedirect(url=None, name=None, params={})
```

```python
self.get("/logout").assertRedirect()
self.get("/logout").assertRedirect("/login")
self.get("/login").assertRedirect(name="profile", params={"user": 1})
```

#### assertCookie

Assert that the request contains the given cookie name and value (if provided).

```python
self.get("/").assertCookie(name, value=None)
```

#### assertPlainCookie

Assert that the request contains the given unencrypted cookie name

```python
self.get("/").assertPlainCookie(name)
```

#### assertCookieExpired

Assert that the request contains the given cookie name and is expired.

```python
self.get("/").assertCookieExpired(name)
```

#### assertCookieMissing

Assert that the request does not contain the given cookie.

```python
self.get("/").assertCookieMissing(name)
```

#### assertSessionHas

Assert that the session contains the given key and value (if provided).

```python
self.get("/").assertSessionHas(name, value=None)
```

#### assertSessionMissing

Assert that the session does not contain the given key.

```python
self.get("/").assertSessionMissing(name)
```

#### assertSessionHasErrors

Assert that the session contains an `errors` key or contains the given list of keys in `errors` key.

```python
self.get("/").assertSessionHasErrors()
```

```python
self.get("/").assertSessionHasErrors(["email", "first_name"])
```

#### assertSessionHasNoErrors

Assert that the session does not contain an `errors` key or that this key is empty.

```python
self.get("/").assertSessionHasNoErrors()
```

#### assertViewIs

Assert that the route returned the given view name.

```python
self.get("/").assertViewIs("app")
```

#### assertViewHas

Assert that view context contains a given data key and value (if provided).

```python
self.get("/").assertViewHas(key, value=None)
```

#### assertViewHasExact

Assert that view context contains exactly the given data keys. It can be a list of keys or a dictionary (here only keys will be verified).

```python
self.get("/").assertViewHasExact(keys)
```

#### assertViewMissing

Assert that given data key is not available in the view context.

```python
self.get("/").assertViewMissing(key)
```

#### assertAuthenticated

Assert that a user is authenticated after the current request.

```python
self.get("/").assertAuthenticated()
```

#### assertGuest

Assert that a user is not authenticated after the current request.

```python
self.get("/").assertGuest()
```

#### assertAuthenticatedAs

Assert that a given user is authenticated after the current request.

```python
self.get("/").assertAuthenticatedAs(user)
```

#### assertHasHttpMiddleware

Assert that the request has the given HTTP middleware. An HTTP middleware class should be provided.

```python
self.get("/").assertHasHttpMiddleware(middleware_class)
```

```python
from app.middleware import MyAppMiddleware
self.get("/").assertHasHttpMiddleware(MyAppMiddleware)
```

#### assertHasRouteMiddleware

Assert that the request has the given route middleware. The registration key of the middleware should be provided.

```python
self.get("/").assertHasRouteMiddleware(middleware_name)
```

```python
# route_middleware = {"web": [SessionMiddleware, VerifyCsrfToken]}
self.get("/").assertHasRouteMiddleware("web")
```

#### assertHasController

Assert that the route used the given controller. A class or a string can be provided. If it's a string it should be formatted as follow `ControllerName@method`.

```python
self.get("/").assertHasController(controller)
```

```python
from app.controllers import WelcomeController
self.get("/").assertHasController(WelcomeController)
self.get("/").assertHasController("WelcomeController@index")
```

#### assertRouteHasParameter

Assert that the route used has the given parameter name.

```python
self.get("/").assertRouteHasParameter(key)
```

#### assertJson

Assert that response is JSON and contains the given data dictionary (if provided). The assertion will pass even if it is not an exact match.

```python
self.get("/").assertJson(data={})
```

```python
self.get("/").assertJson()  # check that response is JSON
self.get("/").assertJson({"key": "value", "other": "value"})
```

#### assertJsonPath

Assert that response is JSON and contains the given path, with eventually the given value if provided. The path can be a dotted path.

```python
self.get("/").assertJsonPath(path, value=None)
```

```python
self.get("/").assertJsonPath("user.profile.name", "John")
```

#### assertJsonExact

Assert that response is JSON and is strictly equal to the given dictionary.

```python
self.get("/").assertJsonExact(data)
```

#### assertJsonCount

Assert that response is JSON and has the given count of keys at root level or at the given key (if provided).

```python
self.get("/").assertJsonCount(count, key=None)
```

#### assertJsonMissing

Assert that response is JSON and does not contain given path. The path can be a dotted path.

```python
self.get("/").assertJsonMissing(path)
```

## Console tests

You can test your [custom commands](/features/commands) running in console with `craft` test helper.

```python
def test_my_command(self):
    self.craft("my_command", "arg1", "arg2").assertSuccess()
```

This will programmatically run the command if it has been registered in your project and assert that no errors has been reported.

### Available Assertions

The following assertions are available when testing command with `craft`.

- [assertSuccess](#assertSuccess)
- [assertHasErrors](#assertHasErrors)
- [assertOutputContains](#assertOutputContains)
- [assertExactOutput](#assertExactOutput)
- [assertOutputMissing](#assertOutputMissing)
- [assertExactErrors](#assertExactErrors)

#### assertSuccess

Assert that command exited with code 0 meaning that it ran successfully.

```python
self.craft("my_command").assertSuccess()
```

#### assertHasErrors

Assert command output has errors.

```python
self.craft("my_command").assertHasErrors()
```

#### assertOutputContains

Assert command output contains the given string.

```python
self.craft("my_command").assertOutputContains(output)
```

#### assertExactOutput

Assert command output to be exactly the same as the given reference output.
Be careful to add eventual `\n` line endings characters when using this assertion method.

```python
self.craft("my_command").assertExactOutput(output)
```

#### assertOutputMissing

Assert command output does not contain the given reference output.

```python
self.craft("my_command").assertOutputMissing(output)
```

#### assertExactErrors

Assert command output has exactly the given errors.

```python
self.craft("my_command").assertExactErrors(errors)
```

## Database tests

By default, your tests are are not ran in isolation from a database point of view. It means that your local database will be modified any time you run your tests and won't be rollbacked at the end of the tests.
While this behaviour might be fine in most case you can learn below how to configure your tests cases to reset the database after each test.

### Resetting The Database After Each Test

If you want to have a clean database for each test you must subclass the `TestCase` class with `DatabaseTransactions` class. Then all your tests will run inside a transaction so any data you create will only exist within the lifecycle of the test. Once the test completes, your database is rolled back to its previous state. This is a perfect way to prevent test data from clogging up your database.

```python
from masonite.tests import TestCase, DatabaseTransactions

class TestSomething(TestCase, DatabaseTransactions):

  connection = "testing"

  def test_can_create_user(self):
      User.create({"name": "john", "email": "john6", "password": "secret"})
```

Note that you can define the `connection` that will be used during testing. This will allow you to select a different database that will be used for testing. Here is a standard exemple of database configuration file that you can use.

```python
# config/database.py
DATABASES = {
    "default": "mysql",
    "mysql": {
        "host": "localhost",
        "driver": "mysql",
        "database": "app",
        "user": "root",
        "password": "",
        "port": 3306
    }
    "testing": {
        "driver": "sqlite",
        "database": "test_database.sqlite3",
    },
}
```

### Available Assertions

Masonite provides several database assertions that can be used during testing.

- [assertDatabaseCount](#assertDatabaseCount)
- [assertDatabaseHas](#assertDatabaseHas)
- [assertDatabaseMissing](#assertDatabaseMissing)
- [assertDeleted](#assertDeleted)
- [assertSoftDeleted](#assertSoftDeleted)

#### assertDatabaseCount

Assert that a table in the database contains the given number of records.

```python
self.assertDatabaseCount(table, count)
```

```python
  def test_can_create_user(self):
      User.create({"name": "john", "email": "john6", "password": "secret"})
      self.assertDatabaseCount("users", 1)
```

#### assertDatabaseHas

Assert that a table in the database contains records matching the given query.

```python
self.assertDatabaseHas(table, query_dict)
```

```python
self.assertDatabaseCount("users", {"name": "John"})
```

#### assertDatabaseMissing

Assert that a table in the database does not contain records matching the given query.

```python
self.assertDatabaseMissing(table, query_dict)
```

```python
self.assertDatabaseMissing("users", {"name": "Jack"})
```

#### assertDeleted

Assert that the given model instance has been deleted from the database.

```python
user=User.find(1)
user.delete()
self.assertDeleted(user)
```

#### assertSoftDeleted

Assert that the given model instance has been [soft deleted](https://orm.masoniteproject.com/models#soft-deleting) from the database.

```python
self.assertSoftDeleted(user)
```

## Helpers

Masonite comes with different helpers that can ease writing tests. Some of them have already been explained in sections above.

- [withExceptionsHandling](#withExceptionsHandling)
- [withoutExceptionsHandling](#withoutExceptionsHandling)
- [withCsrf](#withCsrf)
- [withoutCsrf](#withoutCsrf)
- [withCookies](#withCookies)
- [withHeaders](#withHeaders)
- [fakeTime](#fakeTime)
- [fakeTimeTomorrow](#fakeTimeTomorrow)
- [fakeTimeYesterday](#fakeTimeYesterday)
- [fakeTimeInFuture](#fakeTimeInFuture)
- [fakeTimeInPast](#fakeTimeInPast)
- [restoreTime](#restoreTime)

#### withExceptionsHandling

Enable exceptions handling during testing.

```python
self.withExceptionsHandling()
```

#### withoutExceptionsHandling

Disable exceptions handling during testing.

```python
self.withoutExceptionsHandling()
```

{% hint style="warning" %}
Note that exception handling is disabled by default during testing.
{% endhint %}

#### withCsrf

Enable CSRF protection during testing.

```python
self.withCsrf()
```

#### withoutCsrf

Disable CSRF protection during testing.

```python
self.withoutCsrf()
```

{% hint style="warning" %}
Note that CSRF protection is disabled by default during testing.
{% endhint %}

#### withCookies

Add cookies that will be used in the next request. This method accepts a dictionary of name / value pairs. Cookies dict is reset between each test.

```python
self.withCookies(data)
```

#### withHeaders

Add headers that will be used in the next request. This method accepts a dictionary of name / value pairs. Headers dict is reset between each test.

```python
self.withHeaders(data)
```

#### fakeTime

Set a given pendulum instance to be returned when a `now` (or `today`, `tomorrow` `yesterday`) instance is created. It's really useful during tests to check
timestamps logic.

This allow to control which datetime will be returned to be able to always have an expected behaviour in the tests.

```python
given_date = pendulum.datetime(2021, 2, 5)
self.fakeTime(given_date)
self.assertEqual(pendulum.now(), given_date)
```

{% hint style="warning" %}
When using those helpers you should not forget to reset the default `pendulum` behaviour with `restoreTime()` helper to avoid breaking other tests. It can be done directly in the test or in a `tearDown()` method.
{% endhint %}

#### fakeTimeTomorrow

Set the mocked time as tomorrow. (It's a shortcut to avoid doing `self.fakeTime(pendulum.tomorrow())`).

```python
tomorrow = pendulum.tomorrow()
self.fakeTimeTomorrow()
self.assertEqual(pendulum.now(), tomorrow)
```

#### fakeTimeYesterday

Set the mocked time as yesterday.

#### fakeTimeInFuture

Set the mocked time as an offset of a given unit of time in the future. Unit can be specified among pendulum units: `seconds`, `minutes`, `hours`, `days` (default), `weeks`, `months`, `years`.

```python
self.fakeTimeInFuture(offset, unit="days")
```

```python
real_now = pendulum.now()
self.fakeTimeInFuture(1, "months")
self.assertEqual(pendulum.now().diff(real_now).in_months(), 1)
```

#### fakeTimeInPast

Set the mocked time as an offset of a given unit of time in the past. Unit can be specified among pendulum units: `seconds`, `minutes`, `hours`, `days` (default), `weeks`, `months`, `years`.

```python
self.fakeTimeInPast(offset, unit="days")
```

#### restoreTime

Restore the mocked time behaviour to default behaviour of `pendulum`. When using `fake` time helpers you should not forget to call this helper at the end.

It can be done directly in the test or in a `tearDown()` method.

```python
def tearDown(self):
    super().tearDown()
    self.restoreTime()

def test_creation_date(self):
    self.fakeTimeYesterday()
    # from now on until the end of this unit test, time is mocked and will return yesterday time
```

## Mocks

When it comes to unit testing, you always want to test a unit of your piece of code. Your code might
depends on third party services such as an API and you don't want to call it during your local tests
or in your CI environment. That's when you should use mocking to mock the external parts or the part
you don't want to test.

Masonite comes with some mocking abilities for some of the features relying on third party services.
For other parts or you own code you can use Python mocking abilities provided by `unittest.mock` standard module.

### Masonite features mocks

Masonite tests case have two helpers method `fake()` and `restore()`.

You can mock a Masonite feature by doing `self.fake(feature)` and then restore it to the real feature behaviour
by calling `self.restore(feature)`. When a feature is mocked the real behaviour won't be called, instead
a quick and simple implementation is ran, often offering the ability to inspect and test what happens.

Available features that can be mocked (for now) are:

- [Mail](/features/mail)
- [Notification](/features/notifications)

#### Mocking Mail

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

#### Mocking Notification

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

### Basic Python mocks

For mocking any piece of code in Python you can use the standard `unittest.mock` module. You can find
more information [here](https://docs.python.org/3/library/unittest.mock.html).

Here is basic example

```python
from unittest.mock import patch

with patch("some.module") as SomeClass:
    SomeClass.return_value.my_method.return_value = 0
    self.assertEqual(SomeClass().my_method(), 0)
```

For mocking external HTTP requests you can use the `responses` module.

```python
import responses

@responses.activate
def test_mock_third_party_api(self):
    responses.add(responses.POST, "api.github.com", body=b"Ok")
    # do somehting in your code
    self.assertTrue(responses.assert_call_count("api.github.com", 1))
```

## Extending test response (advanced)

- adding new assertions in HTTPTestResponse
- adding mocking of some feature
