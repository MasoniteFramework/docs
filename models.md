# Models

# Introduction

Out of the box, Masonite comes with the Orator ORM as it



## Has

Sometimes you want to get all records that have some relation. For example you might want to only get all users that have articles:

```python
User.has('articles').get()
```

You may also want a nested relationship. For example, you might want to get all users which have articles that have a logo:

```python
User.has('articles.logo').get()
```

