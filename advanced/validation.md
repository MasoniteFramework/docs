# Validation

## Validation

### Introduction

There are a lot of times when you need to validate incoming input either from a form or from an incoming json request. It is wise to have some form of backend validation as it will allow you. Masonite provides an extremely flexible and fluent way to validate this data.

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
    valid = request.validate(

        validate.required(['user', 'email']),
        validate.accepted(['terms']])

    )

    if valid.errors:
        return request.back().with_errors()
```

This validating will read like "user and email are required and the terms must be accepted" \(more on available rules and what they mean in a bit\)

## Available Rules

|  |  |  |
| :--- | :--- | :--- |
| [accepted](validation.md#accepted) | [is\_in](validation.md#is_in) | [none](validation.md#none) |
| [contains](validation.md#contains) | [isnt](validation.md#isnt) | [numeric](validation.md#numeric) |
| [equals](validation.md#equals) | [json](validation.md#json) | [required](validation.md#required) |
| [greater\_than](validation.md#greater_than) | [length](validation.md#length) | [string](validation.md#string) |
| [in\_range](validation.md#in_range) | [less\_than](validation.md#less_than) | [truthy](validation.md#truthy) |
|  |  | [when](validation.md#when) |

### Accepted

The accepted rule is most useful when seeing if a checkbox has been checked. When a checkbox is submitted it usually has the value of `on` so this rule will check to make sure the value is either on, 1, or yes. 

```python
"""
{
  'terms': 'on'
}
"""
validate.accepted(['terms'])
```

### Contains

This is used to make sure a value exists inside an iterable \(like a list or string\). You may want to check if the string contains the value Masonite for example:

```python
"""
{
  'description': 'Masonite is an amazing framework'
}
"""
validate.contains(['description'], 'Masonite')
```

### Equals

Used to make sure a dictionary value is equal to a specific value

```python
"""
{
  'age': 25
}
"""
validate.equals(['age'], 25)
```

### Greater\_than

This is used to make sure a value is greater than a specific value

```python
"""
{
  'age': 25
}
"""
validate.greater_than(['age'], 18)
```

### In\_range

Used when you need to check if an integer is within a given range of numbers

```python
"""
{
  'attendees': 54
}
"""
validate.in_range(['attendees'], min=24, max=64)
```

### Is\_in

Used to make sure if a value is in a specific value

```python
"""
{
  'age': 5
}
"""
validate.is_in(['age'], [2,4,5])
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
  validate.is_in(['age'], [2,4,5])
)
```

This will produce an error because age it is looking to make sure age **is not in** the list now.

###  Json

Used to make sure a given value is actually a JSON object

```python
"""
{
  'user': 1,
  'payload': '[{"email": "user@email.com"}]'
}
"""
validate.json(['payload'])
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
validate.length(['description'], min=5, max=35)
```

### Less\_than

This is used to make sure a value is less than a specific value

```python
"""
{
  'age': 25
}
"""
validate.less_than(['age'], 18)
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
validate.none(['active'])
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
validate.numeric(['age'])
```

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
validate.string(['email'])
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
validate.truthy(['active'])
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
    validate.less_than(['age'], 18)
).then(
    validate.required(['terms']),
    validate.accepted(['terms'])
)
```

