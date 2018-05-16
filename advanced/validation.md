# Validation

## Introduction

Very often you will find a need to validate forms after you have submitted them. Masonite comes with a very simple and reusable way to validate input data with the Masonite `Validator()` class. In this documentation, we'll talk about how you can create your own validator to use within your project. Masonite uses the `validator.py` library for this feature.

## Getting Started

The best way to create a reusable validator class within your Masonite project is to create a class and inherit from the `Validator` class inside the `masonite.validator` module.

We can make a class called `RegistrationValidator()`, inherit from `masonite.validator.Validator()` and put it inside `app/validators/RegistrationValidator.py` like so:

```python
from masonite.validator import Validator

class RegistrationValidator(Validator):

    def register_form(self):
        pass
```

Awesome! By inheriting from `Validator`, this will add several key methods to our validator class that we'll need to verify our request input which we'll talk about below.

### Starting Our Validations

Since we inherited from `Validator`, we have access to a `validate()` method which we can call with `self.validate()`. Inside this method call, we can use some useful attributes in order to check our request data.

An example of a simple validation will look something like:

```python
from masonite.validator import Validator
from validate import Required, Pattern, Truthy

class RegistrationValidator(Validator):

    def register_form(self):
        self.validate(
            'username': [Required, Pattern('[a-zA-Z0-9]')]
            'is_staff': [Required, Truth()],
            'password': [Required]
        )
```

## Custom Error Messages

By default, Masonite will supply you with a dictionary of errors depending on the validator class. More information on what the default error messages are found below.

Although this is very convenient, you may wish to specify your own error messages. To do so, we can call the `self.messages()` method after we validate. This will look like:

```python
from masonite.validator import Validator
from validate import Required, Pattern, Truthy

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

def show(self, Request):
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

## Validator Options

There are a plethora of options that you can use to validate your forms. In addition to validating your request input, we also get a dictionary of errors. In order to get the errors if a validation fails, we can get use the method:

```python
validate.errors() # {'username': 'must be present'}
```

This method will return a dictionary of errors that will be different depending on the validation class used but this method will return `None` if there are no errors. Below each option will be what the value of `.errors()` will be as well as how you would use them inside Masonite.

### Required

By default, all keys registered for validation are optional. Any key that doesn't exist in the validation will skip any of the missing input data. For example, if a validation is not set for `password` then it will simply not check any validation on that specific request input. In this case, we can leave our `password` validation out entirely.

Unlike other validator classes, this class does not need to be instantiated \(contain parenthesis at the end\). So `Required` is the correct usage and not `Required()`.

#### **Usage**

```python
from validate import Required

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
from validate import Truthy

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
from validate import Equals

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
from validate import Range

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
from validate import Pattern

self.validate({
    'age': [Pattern('\d+')]
})
```

#### **Error**

```python
{"age": ["must match regex pattern \d+"]}

### In
****

This validator checks that the dictionary value is a member of a collection passed to it.


#### Usage

```python
from validate import In

self.validate({
    'username': [In(['Joseph', 'Michael', 'John'])]
})
```

Since Orator returns a collection, we can specify an Orator Collection as well:

```python
from validate import In
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
from validate import Not
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
from validate import InstanceOf

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
from validate import SubclassOf

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
from validate import Length

self.validate({
    'age': [Length(0, maximum=5)]
})
```

#### **Error**

```python
{"age": ["must be at most 5 elements in length"]}
```

