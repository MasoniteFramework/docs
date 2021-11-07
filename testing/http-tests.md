To make a request in your tests, you may use the `get`, `post`, `put`, `patch`, or `delete` methods within your test. These methods do not actually issue a "real" HTTP request to your application. Instead of returning a Masonit class `Response` instance, test request methods return a `HTTPTestResponse`instance, which provides a variety of helpful assertions that allow you to inspect and assert application's responses.

```python
def test_basic_request(self):
    self.get("/").assertOk()
```

## Getting request and response

The request and responses of a test can be fetch by accessing the `request` and `response` attributes.

```python
def test_request_and_response(self):
    request = self.get('/testing').request # <masonite.requests.Request>
    response = self.get('/testing').response # <masonite.response.Response>
```

## Registering routes

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

## Checking if a route exists

To check if a route exists, we can simple use either `get` or `post`:

```python
def test_route_exists(self):
    self.assertTrue(self.get('/testing'))
    self.assertTrue(self.post('/testing'))
```

## Request Headers

You may use the `withHeaders()` method to customize the request's headers before it is sent to the application. This method allows you to add any headers you would like to the request by providing them as a dict:

```python
request = self.withHeaders({"X-TEST": "value"}).get("/").request
self.assertEqual(request.header("X-Test"), "value")
```

## Cookies

You may use the `withCookies()` method to set cookie values before making a request. This method accepts a dictionary of name / value pairs:

```python
self.withCookies({"test": "value"}).get("/").assertCookie("test", "value")
```

## Authentication

If you want to make authenticated requests in your tests, you can use the `actingAs()` method that
takes a given `User` record and authenticate him during the request.

```python
user = User.find(1)
self.actingAs(user).get("/")
```

## CSRF Protection

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

## Exceptions handling

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

## Available Assertions

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
