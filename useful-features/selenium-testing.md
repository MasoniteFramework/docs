# Selenium Testing

## Introduction

Selenium tests are browser tests. With normal unit testing you are usually testing sections of your code like JSON endpoints or seeing a header on a page.

With Selenium testing, you are able to test more advanced features that may require JS handling like what happens when a user clicks a button or submits or a form.

With selenium testing, you will build a bunch of actions for your tests and a browser will open and actually go step by step according to your instructions.

**This package comes shipped with all the required selenium drivers so there is no need for any external selenium server installations.**

## Getting Started

### Installation

First we will need to install the package:

```text
$ pip install masonite-selenium
```

## Creating The TestCase

This package extends the current testing framework so you will add the TestCase to your current unit tests.

Let's build a new `TestCase` and test the homepage that shows when Masonite is first installed:

```text
$ craft test HomePage
```

This will generate a new test that looks like this:

```python
from masonite.testing import TestCase


class TestHomePage(TestCase):

    """All tests by default will run inside of a database transaction."""
    transactions = True

    def setUp(self):
        """Anytime you override the setUp method you must call the setUp method
        on the parent class like below.
        """
        super().setUp()

    def setUpFactories(self):
        """This runs when the test class first starts up.
        This does not run before every test case. Use this method to
        set your database up.
        """
        pass
```

## Building The TestCase

In order to build the selenium test we need to import the class and add it to our `TestCase`.

```python
from masonite.testing import TestCase
from masonite.testing.selenium import SeleniumTestCase


class TestHomePage(TestCase, SeleniumTestCase):

    """All tests by default will run inside of a database transaction."""
    transactions = True

    def setUp(self):
        """Anytime you override the setUp method you must call the setUp method
        on the parent class like below.
        """
        super().setUp()
```

We now have all methods available to us to start building our selenium tests.

## Using a Browser

First before we build our TestCase, we need to specify which browser we want. The current 2 options are: `chrome` and `firefox`.

You can specify which browser to use using the `useBrowser()` method inside the `setUp()` method.

```python
from masonite.testing import TestCase
from masonite.testing.selenium import SeleniumTestCase


class TestHomePage(TestCase, SeleniumTestCase):

    """All tests by default will run inside of a database transaction."""
    transactions = True

    def setUp(self):
        """Anytime you override the setUp method you must call the setUp method
        on the parent class like below.
        """
        super().setUp()
        self.useBrowser('chrome', version='74')
        # self.useBrowser('firefox')
```

If you are using the chrome driver you can optionally specify which version to run.

### Headless

You can optionally specify if you want to run your browsers in a headless state. This means that the browser will not actually open to run tests but will run in the background. This will not effect your tests but is just a preference and usually your tests will run faster.

```python
self.useBrowser('chrome', version='74', headless=True)
```

## Building The Test

Here is a basic example on building a test for the installed homepage:

```python
def test_can_see_homepage(self):
    (self.visit('/')
        .assertSee('Masonite 2.2'))
```

we can then run the test by running:

```text
$ python -m pytest
```

![](../.gitbook/assets/selenium.gif)

## Method Chaining

You can chain all methods together to build up and mock user actions. An example test might look like this:

```python
def test_can_login_to_pypi(self):
    (self.visit('https://pypi.org/')
        .clickLink('Log in')
        .assertSee('Log in to PyPI')
        .text('#username', 'josephmancuso')
        .text('#password', 'secret')
        .submit().assertCantSee('Your projects'))
```

## Selectors

When finding a selector you can use a few symbols to help navigate the page

Take this form for example:

```markup
<form action="/submit">
    <input id="username" type="text" name="username">
    <input id="password" type="password" name="password">

    <button type="submit">Submit</button>
</form>
```

### Selecting by ID

You can select an ID by using the `#` symbol:

```python
(self.visit('/form')
    .text('#username', 'user')
    .text('#password', 'secret')
    .submit())
```

### Selecting by Name

You can select by the name by simply passing in the name value. This will default to the name attribute:

```python
(self.visit('/form')
    .text('username', 'user')
    .text('password', 'secret')
    .submit())
```

### Selecting by Class

```python
(self.visit('/form')
    .text('.username', 'user')
    .text('.password', 'secret')
    .submit())
```

### Selecting by a Unique Attribute

The issue with selecting by a normal selector like an ID or a name is that these could change. This is why you are able to select with a unique attribute name.

You may change your form a bit to do something like this instead:

```markup
<form action="/submit">
    <input selenium="username" type="text" name="username">
    <input selenium="password" type="password" name="password">

    <button type="submit">Submit</button>
</form>
```

You can then tell Masonite what the name of your unique attribute is:

```python
from masonite.testing import TestCase
from masonite.testing.selenium import SeleniumTestCase


class TestHomePage(TestCase, SeleniumTestCase):

    """All tests by default will run inside of a database transaction."""
    transactions = True
    unique_attribute = 'selenium'

    def setUp(self):
        """Anytime you override the setUp method you must call the setUp method
        on the parent class like below.
        """
        super().setUp()
        self.useBrowser('chrome', version='74')
        # self.useBrowser('firefox')
```

and finally you can select by that attribute using the `@` symbol:

```python
(self.visit('/form')
    .text('@username', 'user')
    .text('@password', 'secret')
    .submit())
```

## Available Methods

Below are the available methods you can use to build your tests.

### visit

This method will navigate to a URL

```python
self.visit('/')
```

{% hint style="info" %}
If the URL does not start with http then Masonite will prepend the `APP_URL` environment variable to the front. If this is running inside your Masonite application, you can change this value in your `.env` file.
{% endhint %}

### assertTitleIs

This method will assert that the title is a specific value

```python
(self.visit('https://www.python.org/')
    .assertTitleIs('Welcome'))
```

### assertTitleIsNot

Opposite of `assertTitleIs`.

### assertUrlIs

This method will assert that the current URL is a specific value

```python
(self.visit('https://www.python.org/')
    .assertUrlIs('https://www.python.org/'))
```

### assertSee

Asserts that something is available on the page that the user can see

```python
(self.visit('https://www.python.org/')
    .assertSee('Python'))
```

### assertCanSee

Just an alias for `assertSee`.

### assertCantSee

Opposite of `assertCanSee`. Used to assert that text is not on the page.

### text

Types text into a text box.

```python
(self.visit('https://www.python.org/')
    .text('#username', 'user123')
    .text('#password', 'pass123'))
```

### selectBox

You can choose an option in a select box by its value:

```python
(self.visit('https://www.python.org/')
    .selectBox('#fruit', 'apple'))
```

### check

This will check a checkbox

```python
(self.visit('https://www.python.org/')
    .check('#checkbox'))
```

### resize

Resizes the window based on a width and heigh parameter

```python
(self.visit('https://www.python.org/')
    .resize(800, 600)) # width, height
```

### mazimize

Maximizes the window

```python
(self.visit('https://www.python.org/')
    .maximize())
```

### minimize

Minimizes the window

```python
(self.visit('https://www.python.org/')
    .minimize())
```

### refresh

Refreshes the window

```python
(self.visit('https://www.python.org/')
    .refresh())
```

### back

Navigates backwards

```python
(self.visit('https://www.python.org/')
    .back())
```

### forward

Navigates forwards

```python
(self.visit('https://www.python.org/')
    .forward())
```

### link

Clicks a link on a page

```python
(self.visit('https://www.python.org/')
    .link('#login'))
```

### clickLink

Alias for `link`.

### submit

Submits the current form the last entered element is in

```python
(self.visit('https://www.python.org/')
    .text('#username', 'user123')
    .text('#password', 'pass123')
    .submit())
```

You can also submit another form by entering a selector

```python
(self.visit('https://www.python.org/')
    .text('#username', 'user123')
    .text('#password', 'pass123')
    .submit('#another-form'))
```

### click

Clicks an element

```python
(self.visit('https://www.python.org/')
    .click('#button'))
```

### close

Closes the browser

```python
(self.visit('https://www.python.org/')
    .close())
```

