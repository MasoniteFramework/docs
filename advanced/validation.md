# Validation

## Validation

## Introduction

There are a lot of times when you need to validate incoming input either from a form or from an incoming json request. It is wise to have some form of backend validation as it will allow you to build more secure applications. Masonite provides an extremely flexible and fluent way to validate this data.

Validations are based on rules where you can pass in a key or a list of keys to the rule. The validation will then use all the rules and apply them to the dictionary to validate.

{% hint style="info" %}
You can see a [list of available rules here](validation.md#available-rules).
{% endhint %}

## Validating The Request

Incoming form or JSON data can be validated very simply. All you need to do is import the `Validator` class, resolve it, and use the necessary rule methods.

This whole snippet will look like this in your controller method:

```python
from masonite.validation import Validator

def show(self, request: Request, validate: Validator):
    """
    Incoming Input: {
        'user': 'username123',
        'email': 'user@example.com',
        'terms': 'on'
    }
    """
    errors = request.validate(

        validate.required(['user', 'email']),
        validate.accepted('terms')

    )

    if errors:
        request.session.flash('errors', errors)
        return request.back()
```

This validating will read like "user and email are required and the terms must be accepted" \(more on available rules and what they mean in a bit\)

{% hint style="info" %}
Note you can either pass in a single value or a list of values
{% endhint %}

## Creating a Rule

Sometimes you may cross a time where you need to create a new rule that isn't available in Masonite or there is such a niche use case that you need to build a rule for.

In this case you can create a new rule.

### Rule Command

You can easily create a new rule boiler plate by running:

{% tabs %}
{% tab title="terminal" %}
```bash
$ craft rule equals_masonite
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
There is no particular reason that rules are lowercase class names. The main reason it is improves readability when you end up using it as a method if you choose to register the rule with the validation class like you will see below.
{% endhint %}

This will create a boiler plate rule inside app/rules/equals\_masonite.py that looks like:

```python
class equals_masonite(BaseValidation):
    """A rule_name validation class
    """

    def passes(self, attribute, key, dictionary):
        """The passing criteria for this rule.

        ...
        """
        return attribute

    def message(self, key):
        """A message to show when this rule fails

        ...
        """
        return '{} is required'.format(key)

    def negated_message(self, key):
        """A message to show when this rule is negated using a negation rule like 'isnt()'

        ...
        """
        return '{} is not required'.format(key)
```

### Constructing our Rule

Our rule class needs 3 methods that you see when you run the rule command, a `passes`, `message` and `negated_message` methods.

**Passes Method**

The passes method needs to return some kind of boolean value for the use case in which this rule passes.

For example if you need to make a rule that a value always equals Masonite then you can make the method look like this:

```python
def passes(self, attribute, key, dictionary):
    """The passing criteria for this rule.

    ...
    """
    return attribute == 'Masonite'
```

When validating a dictionary like this:

```python
{
  'name': 'Masonite'
}
```

then

* the `attribute` will be the value \(`Masonite`\)
* the `key` will be the dictionary key \(`name`\)
* the `dictionary` will be the full dictionary in case you need to do any additional checks.

**Message method**

The message method needs to return a string used as the error message. If you are making the rule above then our rule may so far look something like:

```python
def passes(self, attribute, key, dictionary):
    """The passing criteria for this rule.

    ...
    """
    return attribute == 'Masonite'

def message(self, key):
    return '{} must be equal to Masonite'.format(key)
```

**Negated Message**

The negated message method needs to return a message when this rule is negated. This will basically be a negated statement of the `message` method:

```python
def passes(self, attribute, key, dictionary):
    """The passing criteria for this rule.

    ...
    """
    return attribute == 'Masonite'

def message(self, key):
    return '{} must be equal to Masonite'.format(key)

def negated_message(self, key):
    return '{} must not be equal to Masonite'.format(key)
```

### Registering our Rule

Now the rule is created we can use it in 1 of 2 ways.

**Importing our rule**

We can either import directly into our controller method:

```python
from masonite.validation import Validator
from app.rules.equals_masonite import equals_masonite

def show(self, request: Request, validate: Validator):
    """
    Incoming Input: {
        'user': 'username123',
        'company': 'Masonite'
    }
    """
    valid = request.validate(

        validate.required(['user', 'company']),
        equals_masonite('company')

    )
```

or we can register our rule and use it with the Validator class as normal.

**Register the rule**

In any service provider's boot method \(preferably a provider where `wsgi=False` to prevent it from running on every request\) we can register our rule with the validator class.

If you don't have a provider yet we can make one specifically for adding custom rules:

{% tabs %}
{% tab title="terminal" %}
```bash
$ craft provider RuleProvider
```
{% endtab %}
{% endtabs %}

Then inside this rule provider's boot method we can resolve and register our rule. This will look like:

```python
from app.rules.equals_masonite import equals_masonite
from masonite.validation import Validator

class RuleProvider(ServiceProvider):
    """Provides Services To The Service Container
    """

    wsgi = False

    ...

    def boot(self, validator: Validator):
        """Boots services required by the container
        """

        validator.register(equals_masonite)
```

Now instead of importing the rule we can just use it as normal:

```python
from masonite.validation import Validator

def show(self, request: Request, validate: Validator):
    """
    Incoming Input: {
        'user': 'username123',
        'company': 'Masonite'
    }
    """
    valid = request.validate(

        validate.required(['user', 'company']),
        validate.equals_masonite('company')

    )
```

notice we called the method as if it was apart of the validator class this whole time.

{% hint style="info" %}
Registering rules is especially useful when creating packages for Masonite that contain new rules.
{% endhint %}

## Using The Validator Class

In addition to validating the request class we can also use the validator class directly. This is useful if you need to validate your own dictionary:

```python
from masonite.validation import Validator

def show(self, validator: Validator):
    """
    Incoming Input: {
        'user': 'username123',
        'company': 'Masonite'
    }
    """
    valid = validator.validate({
        'user': 'username123',
        'company': 'Masonite'
    },
        validate.required(['user', 'company']),
        validate.equals_masonite('company')
    )
```

Just put the dictionary as the first argument and then each rule being its own argument.

## Using The Decorator

Masonite validation has a convenient decorator you can use on your controller methods. This will prevent the controller method being hit all together if validation isn't correct:

```python
from masonite.validation.decorators import validate
from masonite.validation import required

@validate(required('name'))
def show(self, view: View):
  return view.render(..)
```

This will return a JSON response. You can also choose where to redirect back to:

```python
from masonite.validation.decorators import validate
from masonite.validation import required

@validate(required('name'), redirect='/login')
def show(self, view: View):
  return view.render(..)
```

As well as redirect back to where you came from \(if you use the `{{ back() }}` template helper\)

```python
from masonite.validation.decorators import validate
from masonite.validation import required

@validate(required('name'), back=True)
def show(self, view: View):
  return view.render(..)
```

> Both of these redirections will redirect with errors and input. So you can use the `{{ old() }}` template helper to get previous input.

## Rule Enclosures

Rule enclosures are self contained classes with rules. You can use these to help reuse your validation logic. For example if you see you are using the same rules often you can use an enclosure to always keep them together and reuse them throughout your code base.

### Rule Enclosure Command

You can create a rule enclosure by running:

```text
$ craft rule:enclosure AcceptedTerms
```

You will then see a file generated like this inside app/rules:

```python
from masonite.validation import RuleEnclosure

...

class AcceptedTerms(RuleEnclosure):

    def rules(self):
        """ ... """
        return [
            # Rules go here
        ]
```

### Creating the Rule Enclosure

You can then fill the list with rules:

```python
from masonite.validation import required, accepted

class LoginForm(RuleEnclosure):

    def rules(self):
        """ ... """
        return [
            required(['email', 'terms']),
            accepted('terms')
        ]
```

You can then use the rule enclosure like this:

```python
from app.rules.LoginForm import AcceptedTerms

def show(self, request: Request):
    """
    Incoming Input: {
        'user': 'username123',
        'email': 'user@example.com',
        'terms': 'on'
    }
    """
    errors = request.validate(AcceptedTerms)

    if errors:
        request.session.flash('errors', errors)
        return request.back()
```

You can also use this in addition to other rules:

```python
from app.rules.LoginForm import AcceptedTerms
from masonite.validations import email

def show(self, request: Request):
    """
    Incoming Input: {
        'user': 'username123',
        'email': 'user@example.com',
        'terms': 'on'
    }
    """
    errors = request.validate(
        AcceptedTerms,
        email('email')
    )

    if errors:
        request.session.flash('errors', errors)
        return request.back()
```

## Message Bag

Working with errors may be a lot especially if you have a lot of errors which results in quite a big dictionary to work with.

Because of this, Masonite Validation comes with a `MessageBag` class which you can use to wrap your errors in. This will look like this:

```python
from masonite.validation import MessageBag
# ...
def show(self, request: Request):
    errors = request.validate(
        email('email')
    )
    errors = MessageBag(errors)
```

### Getting All Errors:

You can easily get all errors using the `all()` method:

```python
errors = MessageBag(errors)
errors.all()
"""
{
  'email': ['Your email is required'],
  'name': ['Your name is required']
}
"""
```

### Checking for any errors

```python
errors = MessageBag(errors)
errors.any() #== True
```

### Checking if the bag is Empty

This is just the opposite of the `any()` method.

```python
errors = MessageBag(errors)
errors.empty() #== False
```

### Checking For a Specific Error

```python
errors = MessageBag(errors)
errors.has('email') #== True
```

### Getting the first Key:

```python
errors = MessageBag(errors)
errors.all()
"""
{
  'email': ['Your email is required'],
  'name': ['Your name is required']
}
"""
errors.first()
"""
{
  'email': ['Your email is required']
}
"""
```

### Getting the Number of Errors:

```python
errors = MessageBag(errors)
errors.count() #== 2
```

### Converting to JSON

```python
errors = MessageBag(errors)
errors.json()
"""
'{"email": ["Your email is required"],"name": ["Your name is required"]}'
"""
```

### Get the Amount of Messages:

```python
errors = MessageBag(errors)
errors.amount('email') #== 1
```

### Get the Messages:

```python
errors = MessageBag(errors)
errors.amount('email')
"""
['Your email is required']
"""
```

### Get the Errors

```python
errors = MessageBag(errors)
errors.errors()
"""
['email', 'name']
"""
```

### Get all the Messages:

```python
errors = MessageBag(errors)
errors.messages()
"""
['Your email is required', 'Your name is required']
"""
```

### Merge a Dictionary

You can also merge an existing dictionary into the bag with the errors:

```python
errors = MessageBag(errors)
errors.merge({'key': 'value'})
```

## Template Helper

You can use the `bag()` template helper which will contain the list of errors. Inside an HTML template you can do something like this:

```markup
@if(bag().any())
    <div class="bg-yellow-200 text-yellow-800 px-4 py-2">
        <ul>
            @for message in bag().messages()
            <li>{{ message }}</li>
            @endfor
        </ul>
    </div>
@endif
```

This will give you all the errors inside each list.

## Nested Validations

Sometimes you will need to check values that aren't on the top level of a dictionary like the examples shown here. In this case we can use dot notation to validate deeper dictionaries:

```python
"""
{
  'domain': 'http://google.com',
  'email': 'user@example.com'
  'user': {
     'id': 1,
     'email': 'user@example.com',
     'status': {
         'active': 1,
         'banned': 0
     }
  }
}
"""
errors = request.validate(

    validate.required('user.email'),
    validate.truthy('user.status.active')

)
```

notice the dot notation here. Each `.` being a deeper level to the dictionary.

### Nested Validations With Lists

Sometimes your validations will have lists and you will need to ensure that each element in the list validates. For example you want to make sure that a user passes in a list of names and ID's.

For this you can use the \* asterisk to validate these:

```python
"""
{
  'domain': 'http://google.com',
  'email': 'user@example.com'
  'user': {
     'id': 1,
     'email': 'user@example.com',
     'addresses': [{
         'id': 1, 'street': 'A Street',
         'id': 2, 'street': 'B Street'
     }]
  }
}
"""
```

Here is an example to make sure that street is a required field:

```python
errors = request.validate(

    validate.required('user.addresses.*.street'),
    validate.integer('user.addresses.*.id'),

)
```

## Custom Messages

All errors returned will be very generic. Most times you will need to specify some custom error that is more tailored to your user base.

Each rule has a messages keyword arg that can be used to specify your custom errors.

```python
"""
{
  'terms': 'off',
  'active': 'on',
}
"""
validate.accepted(['terms', 'active'], messages = {
    'terms': 'You must check the terms box on the bottom',
    'active': 'Make sure you are active'
})
```

Now instead of returning the generic errors, the error message returned will be the one you supplied.

{% hint style="info" %}
Leaving out a message will result in the generic one still being returned for that value.
{% endhint %}

## Exceptions

By default, Masonite will not throw exceptions when it encounters failed validations. You can force Masonite to raise a `ValueError` when it hits a failed validation:

```python
"""
{
  'domain': 'http://google.com',
  'email': 'user@example.com'
  'user': {
     'id': 1,
     'email': 'user@example.com',
     'status': {
         'active': 1,
         'banned': 0
     }
  }
}
"""
errors = request.validate(

    validate.required('user.email', raises=True),
    validate.truthy('user.status.active')

)
```

Now if the required rule fails it will throw a `ValueError`. You can catch the message like so:

```python
try:
    errors = request.validate(

        validate.required('user.email', raises=True),
        validate.truthy('user.status.active')

    )
except ValueError as e:
    str(e) #== 'user.email is required'
```

### Custom Exceptions

You can also specify which exceptions should be thrown with which key being checked by using a dictionary:

```python
try:
    errors = request.validate(

        validate.required(['user.email', 'user.id'], raises={
            'user.id': AttributeError,
            'user.email': CustomException
        }),

    )
except AttributeError as e:
    str(e) #== 'user.id is required'
except CustomException as e:
    str(e) #== 'user.email is required'
```

All other rules within an explicit exception error will throw the `ValueError`.

## String Validation

In addition to using the methods provided below, you can also use each one as a pipe delimitted string. For example these two validations are identical:

```python
# Normal
errors = request.validate(
  validate.required(['email', 'username', 'password', 'bio']),
  validate.accepted('terms'),
  validate.length('bio', min=5, max=50),
  validate.strong('password')
)

# With Strings
errors = request.validate({
  'email': 'required',
  'username': 'required',
  'password': 'required|strong',
  'bio': 'required|length:5..50'
  'terms': 'accepted'
})
```

These rules are identical so use whichever feels more comfortable.

## Available Rules

|  |  |  |
| :--- | :--- | :--- |
| [accepted](validation.md#accepted) | [is\_in](validation.md#is_in) | [truthy](validation.md#truthy) |
| [active\_domain](validation.md#active_domain) | [isnt](validation.md#isnt) | [when](validation.md#when) |
| [after\_today](validation.md#after_today) | [is\_past](validation.md#is_past) | [timezone](validation.md#timezone) |
| [before\_today](validation.md#before_today) | [is\_future](validation.md#is_future) | [phone](validation.md#phone) |
| [contains](validation.md#contains) | [json](validation.md#json) | [strong](validation.md#strong) |
| [email](validation.md#email) | [less\_than](validation.md#less_than) |  |
| [equals](validation.md#equals) | [length](validation.md#length) |  |
| [exists](validation.md#exists) | [none](validation.md#none) |  |
| [greater\_than](validation.md#greater_than) | [numeric](validation.md#numeric) |  |
| [in\_range](validation.md#in_range) | [required](validation.md#required) |  |
| [ip](validation.md#ip) | [string](validation.md#string) |  |

### Accepted

The accepted rule is most useful when seeing if a checkbox has been checked. When a checkbox is submitted it usually has the value of `on` so this rule will check to make sure the value is either on, 1, or yes.

```python
"""
{
  'terms': 'on'
}
"""
validate.accepted('terms')
```

### Active\_domain

This is used to verify that the domain being passed in is a DNS resolvable domain name. You can also do this for email addresses as well. The preferred search is domain.com but Masonite will strip out `http://`, `https://` and `www` automatically for you.

```python
"""
{
  'domain': 'http://google.com',
  'email': 'user@example.com'
}
"""
validate.active_domain(['domain', 'email'])
```

### After\_today

Used to make sure the date is a date after today. In this example, this will work for any day that is 2019-10-21 or later.

```python
"""
{
  'date': '2019-10-20', # Or date in the future
}
"""
validate.after_today('date')
```

You may also pass in a timezone for this rule:

```python
"""
{
  'date': '2019-10-20', # Or date in the future
}
"""
validate.after_today('date', tz='America/New_York')
```

### Before\_today

Used to make sure the date is a date before today. In this example, this will work for any day that is 2019-10-19 or earlier.

```python
"""
{
  'date': '2019-10-20', # Or date in the past
}
"""
validate.before_today('date')
```

You may also pass in a timezone for this rule:

```python
"""
{
  'date': '2019-10-20', # Or date in the past
}
"""
validate.before_today('date', tz='America/New_York')
```

### Contains

This is used to make sure a value exists inside an iterable \(like a list or string\). You may want to check if the string contains the value Masonite for example:

```python
"""
{
  'description': 'Masonite is an amazing framework'
}
"""
validate.contains('description', 'Masonite')
```

### Confirmed

This rule is used to make sure a key is "confirmed". This is simply a `key_confirmation` representation of the key.

For example, if you need to confirm a `password` you would set the password confirmation to `password_confirmation`.

```python
"""
{
  'password': 'secret',
  'password_confirmation': 'secret'
}
"""
validate.confirmed('password')
```

### Does\_not

Used for running a set of rules when a set of rules does not match. Has a `then()` method as well. Can be seen as the opposite of when.

```python
"""
{
  'age': 15,
  'email': 'user@email.com',
  'terms': 'on'
}
"""
validate.does_not(
    validate.exists('user')
).then(
    validate.accepted('terms'),
)
```

### Equals

Used to make sure a dictionary value is equal to a specific value

```python
"""
{
  'age': 25
}
"""
validate.equals('age', 25)
```

### Email

This is useful for verifying that a value is a valid email address

```python
"""
{
  'domain': 'http://google.com',
  'email': 'user@example.com'
}
"""
validate.email('email')
```

### Exists

Checks to see if a key exists in the dictionary.

```python
"""
{
  'email': 'user@example.com',
  'terms': 'on'
  'age': 18
}
"""
validate.exists('terms')
```

This is good when used with the when rule:

```python
"""
{
  'email': 'user@example.com',
  'terms': 'on'
  'age': 18
}
"""
validate.when(
    validate.exists('terms')
).then(
    validate.greater_than('age', 18)
)
```

### Greater\_than

This is used to make sure a value is greater than a specific value

```python
"""
{
  'age': 25
}
"""
validate.greater_than('age', 18)
```

### In\_range

Used when you need to check if an integer is within a given range of numbers

```python
"""
{
  'attendees': 54
}
"""
validate.in_range('attendees', min=24, max=64)
```

### Ip

You can also check if the input is a valid IPv4 address:

```python
"""
{
  'address': '78.281.291.8'
}
"""
validate.ip('address')
```

### Is\_in

Used to make sure if a value is in a specific value

```python
"""
{
  'age': 5
}
"""
validate.is_in('age', [2,4,5])
```

notice how 5 is in the list

### Isnt

This will negate all rules. So if you need to get the opposite of any of these rules you will add them as rules inside this rule.

For example to get the opposite if `is_in` you will do:

```python
"""
{
  'age': 5
}
"""
validate.isnt(
  validate.is_in('age', [2,4,5])
)
```

This will produce an error because age it is looking to make sure age **is not in** the list now.

### Is\_future

Checks to see the date and time passed is in the future. This will pass even if the datetime is 5 minutes in the future.

```python
"""
{
  'date': '2019-10-20', # Or date in the future
}
"""
validate.is_future('date')
```

You may also pass in a timezone for this rule:

```python
"""
{
  'date': '2019-10-20', # Or date in the future
}
"""
validate.is_future('date', tz='America/New_York')
```

### Is\_past

Checks to see the date and time passed is in the past. This will pass even if the datetime is 5 minutes in the past.

```python
"""
{
  'date': '2019-10-20', # Or date in the future
}
"""
validate.is_past('date')
```

You may also pass in a timezone for this rule:

```python
"""
{
  'date': '2019-10-20', # Or date in the future
}
"""
validate.is_past('date', tz='America/New_York')
```

### Json

Used to make sure a given value is actually a JSON object

```python
"""
{
  'user': 1,
  'payload': '[{"email": "user@email.com"}]'
}
"""
validate.json('payload')
```

### Length

Used to make sure a string is of a certain length

```python
"""
{
  'user': 1,
  'description': 'this is a long description'
}
"""
validate.length('description', min=5, max=35)
```

### Less\_than

This is used to make sure a value is less than a specific value

```python
"""
{
  'age': 25
}
"""
validate.less_than('age', 18)
```

### None

Used to make sure the value is None

```python
"""
{
  'age': 25,
  'active': None
}
"""
validate.none('active')
```

### Numeric

Used to make sure a value is a numeric value

```python
"""
{
  'age': 25,
  'active': None
}
"""
validate.numeric('age')
```

### One\_of

Sometimes you will want only one of several fields to be required. At least one of them need to be required.

```python
"""
{
  'user': 'Joe',
  'email': 'user@example.com,
  'phone': '123-456-7890'
}
"""
validate.one_of(['user', 'accepted', 'location'])
```

This will pass because at least 1 value has been found: `user`.

### Phone

You can also use the phone validator to validate the most common phone number formats:

```python
"""
{
  'phone': '876-827-9271'
}
"""
validate.phone('phone', pattern='123-456-7890')
```

The available patterns are:

* `123-456-7890`
* `(123)456-7890`

### Required

Used to make sure the value is actually available in the dictionary. This will add errors if the key is not present

```python
"""
{
  'age': 25,
  'email': 'user@email.com'
}
"""
validate.required(['age', 'email'])
```

### String

Used to make sure the value is a string

```python
"""
{
  'age': 25,
  'email': 'user@email.com'
}
"""
validate.string('email')
```

### Strong

The strong rule is used to make sure a string has a certain amount of characters required to be considered a "strong" string.

This is really useful for passwords when you want to make sure a password has at least 8 characters, have at least 2 uppercase letters and at least 2 special characters.

```python
"""
{
  'email': 'user@email.com'
  'password': 'SeCreT!!'
}
"""
validate.strong('password', length=8, special=2, uppercase=3)
```

### Timezone

You can also validate that a value passed in a valid timezone

```python
"""
{
  'timezone': 'America/New_York'
}
"""
validate.timezone('timezone')
```

### Truthy

Used to make sure a value is a truthy value. This is anything that would pass in a simple if statement.

```python
"""
{
  'active': 1,
  'email': 'user@email.com'
}
"""
validate.truthy('active')
```

### When

Conditional rules. This is used when you want to run a specific set of rules only if a first set of rules succeeds.

For example if you want to make terms be accepted ONLY if the user is under 18

```python
"""
{
  'age': 15,
  'email': 'user@email.com',
  'terms': 'on'
}
"""
validate.when(
    validate.less_than('age', 18)
).then(
    validate.required('terms'),
    validate.accepted('terms')
)
```

