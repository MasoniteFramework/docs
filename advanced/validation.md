# Validation

## Introduction

Very often you will find a need to validate forms after you have submitted them. Masonite comes with a very simple and reusable way to validate input data with the Masonite `Validator()` class. In this documentation, we'll talk about how you can create your own validator to use within your project. Masonite uses the `validator.py` library for this feature.

## Getting Started

The best way to create a reusable validator class within your Masonite project is to create a class and inherit from the `Validator` class inside the `masonite.validator` module.

We can make a class called `RegistrationValidator()`, inherit from `masonite.validator.Validator()` and put it inside `app/validators/RegistrationValidator.py` like so:

{% code-tabs %}
{% code-tabs-item title="app/validators/RegistrationValidator.py" %}
```python
from masonite.validator import Validator

class RegistrationValidator(Validator):

    def register_form(self):
        pass
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can put this file wherever you want but we will put it here for the purposes of this documentation.

Awesome! By inheriting from `Validator`, this will add several key methods to our validator class that we'll need to verify our request input which we'll talk about below.

### Starting Our Validations

Since we inherited from `Validator`, we have access to a `validate()` method which we can call with `self.validate()`. Inside this method call, we can use some useful attributes in order to check our request data.

An example of a simple validation will look something like:

```python
from masonite.validator import Validator
from validator import Required, Pattern, Truthy

class RegistrationValidator(Validator):

    def register_form(self):
        self.validate({
            'username': [Required, Pattern('[a-zA-Z0-9]')]
            'is_staff': [Required, Truth()],
            'password': [Required]
        })
```

## Custom Error Messages

By default, Masonite will supply you with a dictionary of errors depending on the validator class. More information on what the default error messages are found below.

Although this is very convenient, you may wish to specify your own error messages. To do so, we can call the `self.messages()` method after we validate. This will look like:

```python
from masonite.validator import Validator
from validator import Required, Pattern, Truthy

class RegistrationValidator(Validator):

    def register_form(self):
        self.validate({
            'username': [Required, Pattern('[a-zA-Z0-9]')]
            'is_staff': [Required, Truth()],
            'password': [Required]
        })

        self.messages({
            'username': 'The username is required!',
            'password': 'Make this password top secret!'
        })
```

This will change the default error messages to the defaults provided. If you do not specify a custom error message for a validation field then the default one will display as usual.

### Requests

In order to check the input data we receive from a request, such as a form submission, we can use this validator like so:

```python
from app.validators.RegistrationValidator import RegistrationValidator

def show(self, request: Request):
    validate = RegistrationValidator(Request)
    validate.register_form()
    validate.check() # returns True or False
    validate.errors() # returns a dictionary of errors if any
```

Notice that we pass in the request inside the constructor of our `RequestValidator` class. Our class is using the constructor inherited from Masonite's `Validator` class.

### Non Requests

Sometimes we may wish to use our validator class without request data. In order to do this we can just pass a dictionary of values into our `.check()` method like so:

```python
from app.validators.RegistrationValidator import RegistrationValidator

def show(self):
    validate = RegistrationValidator()
    validate.register_form()
    validate.check({'username': 'John'}) # returns True or False
    validate.errors() # returns a dictionary of errors if any
```

Notice how we passed a dictionary into our `.check()` method here and didn't pass the request object in the constructor.

## Casting Values

Sometimes you might want to cast a certain value for the purposes of validation. For example you may want to accept a string like `California,Florida,Ohio` but wan't to treat it as a list for the purposes of validation. We might want to make it a list to see if there are between 1 and 3 of them.

To do this simply requires validation casting. When validating, Masonite Validations will look for a cast\_x method. x is the name of the input you are validating. So for example we may have an input like this:

```python
request.input('countries') # 'California,Florida,Ohio'
```

```python
from masonite.validator import Validator
from validator import Required, Length, Truthy

class CountryValidation(Validator):

    def validate_countries(self):
        return self.validate({
            'countries': [Required, Length(1, 3)]
        })

    def cast_countries(self, countries):
        return countries.split(',')
```

When validating, the validator will call this cast\_countries method and use the return value as the value to validate. This validation will work because it will split the countries into a list and turn:

```python
'California,Florida,Ohio'
```

into this:

```python
['California', 'Florida', 'Ohio']
```

### Getting Casted Values

You can also use this validator class to get the value that the validator is using. In other words, if there is a cast method, it will get the casted value. If there is no cast method then it will get the raw value supplied to it.

```python
from app.validators import CountryValidation

def show(self, request: Request):
    request.input('countries') # 'California,Florida,Ohio'
    validation = CountryValidation(Request).validate_countries()

    request.input('countries') # 'California,Florida,Ohio'
    validation.get('countries') # ['California', 'Florida', 'Ohio']
```

## Validation Helper

If a full validation class is a bit too much for you then you can use a smaller helper function. You may choose this option if you are building a simple RESTful API endpoint and just want some quick validation.

This helper function signature looks like:

```python
from masonite.helpers import validate

validation = validate({rules}, {input}, {messages})
```

Messages are optional. If left out it will use the default error messages.

A full implementation looks like:

```python
from masonite.helpers import validate

validation = validate(
              {'id': [Required]}, # rules
              {'name': '1,2'}, # input
              {'id': 'ID is required!'} # custom error messages
            )
```

## Validator Options

There are a plethora of options that you can use to validate your forms. In addition to validating your request input, we also get a dictionary of errors. In order to get the errors if a validation fails, we can use the method:

```python
validate.errors() # {'username': 'must be present'}
```

This method will return a dictionary of errors that will be different depending on the validation class used but this method will return `None` if there are no errors. Below each option will be what the value of `.errors()` will be as well as how you would use them inside Masonite.

### Required

By default, all keys registered for validation are optional. Any key that doesn't exist in the validation will skip any of the missing input data. For example, if a validation is not set for `password` then it will simply not check any validation on that specific request input. In this case, we can leave our `password` validation out entirely.

Unlike other validator classes, this class does not need to be instantiated \(contain parenthesis at the end\). So `Required` is the correct usage and not `Required()`.

#### **Usage**

```python
from validator import Required

self.validate({
    'username': [Required]
})
```

#### **Error**

```python
{"username": ["must be present"]}
```

### Truthy

The `Truthy()` validator class will check whatever is truthy to Python. This includes True, non-0 integers, non-empty lists, and strings

#### **Usage**

```python
from validator import Truthy

self.validate({
    'username': [Truthy()]
})
```

#### **Error**

```python
{"username": ["must be True-equivalent value"]}
```

### Equals

This validator will check that a value is equal to the value given.

#### **Usage**

```python
from validator import Equals

self.validate({
    'username': [Equals('Joseph')]
})
```

#### **Error**

```python
{"username": ["must be equal to 'Joseph'"]}
```

{% hint style="warning" %}
**Note that all request input data will be a string. so make sure you use** `Equals('1')` **and not** `Equals(1)`**. Just be sure to maintain the data type in your validation.**
{% endhint %}

### Range

This validator checks that the dictionary value falls inclusively between the start and end values passed to it.

```python
from validator import Range

self.validate({
    'age': [Range(1, 100)]
})
```

#### **Error**

```python
{"age": ["must fall between 1 and 100"]}
```

### Pattern

The Pattern validator checks that the dictionary value matches the regex pattern that was passed to it.

#### **Usage**

```python
from validator import Pattern

self.validate({
    'age': [Pattern('\d+')]
})
```

#### **Error**

```python
{"age": ["must match regex pattern \d+"]}
```

### In

This validator checks that the dictionary value is a member of a collection passed to it.

**Usage**

```python
from validator import In 

self.validate({ 'username': [In(['Joseph', 'Michael', 'John'])]})
```

Since Orator returns a collection, we can specify an Orator Collection as well:

```python
from validator import In
from config import database
users = db.table('users').select('name').get()

self.validate({
    'username': [In([users])]
})
```

#### **Error**

```python
{"age": ["must be one of <collection here>"]}
```

### Not

This validator negates a validator that is passed to it and checks the dictionary value against that negated validator.

#### **Usage**

```python
from validator import Not
from config import database
users = db.table('users').select('name').get()

self.validate({
    'age': [Not(In(users))]
})
```

#### **Error**

```python
{"age": ["must be one of <collection here>"]}
```

### InstanceOf

This validator checks that the dictionary value is an instance of the base class passed to it, or an instance of one of its subclasses.

#### **Usage**

```python
from validator import InstanceOf

self.validate({
    'age': [InstanceOf(basestring)]
})
```

#### **Error**

```python
{"age": ["must be an instance of basestring or its subclasses"]}
```

### SubclassOf

This validator checks that the dictionary value inherits from the base class passed to it. To be clear, this means that the dictionary value is expected to be a class, not an instance of a class.

#### **Usage**

```python
from validator import SubclassOf

self.validate({
    'age': [SubclassOf(str)]
})
```

#### **Error**

```python
{"age": ["must be a subclass of str"]}
```

### Length

This validator checks that value the must have at least minimum elements and optionally at most maximum elements.

#### **Usage**

```python
from validator import Length

self.validate({
    'age': [Length(0, maximum=5)]
})
```

#### **Error**

```python
{"age": ["must be at most 5 elements in length"]}
```

