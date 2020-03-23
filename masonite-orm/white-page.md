# White Page

## Outline ORM White Paper

* [x] Query Builder
* [x] Model
* [x] Schema
* [x] Grammar class
* [x] Relationships

> **This project is not currently in Masonite but is in development. This article is designed to be a learning document to aid in those who wish to contribute to the project. This article will be updated when concepts are added to the ORM project or when concepts change. It is still a young project so concepts may change but the basic foundation is in place and the foundation you read here likely will not change. You can contribute to the project at the** [**Masonite ORM Repo**](https://github.com/MasoniteFramework/orm)

## The Flow

I will discuss the flow at a high level first and then can talk about each part separarely.

First, everything starts with a model. You will import the model into your application and run some methods on it. Most of these methods simply return a instance of a query builder to kick off the flow.

For example,

```python
user = User
user #== <class User>
user.where('id', 1) #== <masonite.orm.Querybuilder object>
```

Since it returns a query builder we can simply build up this class and chain on a whole bunch of methods:

```python
user.where('id', 1).where('active', 1) #== <masonite.orm.Querybuilder object>
```

Finally when we are done building a query we will call a `.get()`:

```python
user.select('id').where('id', 1).where('active', 1).get() #== <masonite.orm.Collection object>
```

When you call get, the query builder will pass everything you built up \(1 select, 2 where statements\) and pass those into a grammar class. The grammar class is responsible for looping through the 3 statements and compiling them into a query that will run. So the grammar class will compile a query that looks like this:

```text
SELECT `id` FROM `users` WHERE `id` = '1' AND `active` = 1
```

Once we get the query we can then pass the query into the connection class which will connect to the MySQL database to send the query.

We will then get back a dictionary from the query and "hydrate" the original model. When you hydrate a model it simply means we set the dictionary result into the model so when we access something like `user.name` we will get the name of the user. Think of it as loading the dictionary into the class to be fetched later.

## Grammar Classes

Grammar classes are classes which are responsible for the compiling of attributes into a SQL statement. The SQL statement will then be given to the connection class to make the database call and return the result. Again the grammar class is only responsible for compiling the query into a string.

The grammar class will be responsible for both SQL and Qmark. SQL looks like this:

```text
SELECT * FROM `users` where `age` = '18'
```

Qmark is this:

```text
SELECT * FROM `users` where `age` = '?'
```

Qmark queries will then be passed off to the connection class with a tuple of bindings like `(18,)`. This helps protect against SQL injection.

**Any values should be able to be qmarked**. This is done inside the grammar class by replacing the value with a `'?'` and then adding the value to the bindings.

The grammar class is also really an abstraction as well. All the heavy lifting is done inside the `BaseGrammar` class. Child classes \(like `MySQLGrammar` and `PostgresGrammar`, etc\) really just contain the formatting of the sql strings.

Almost all SQL is bascially the same but with slightly different formats.

For example, MySQL has this format for select statements with a limit:

```text
SELECT * from `users` LIMIT 1
```

But Microsoft SQL Server has this:

```text
SELECT TOP 1 * from `users`
```

Notice the SQL is bascially the same but the limit statement is in a different spot in the SQL.

We can accomplish this by specifying the format order so we can swap the placement. We do this by using Python keyword string interpolation. For example let's break down a more low level on how we can accomplish this:

Here is the MySQL grammar class select statement structure. I will simplify this for the sake of explanation but just know this also contains the formatting for joins, group by's in the form of `{joins}`, `{group_by}` etc:

MySQL:

```python
def select_format(self):
    return "SELECT {columns} FROM {table} {limit}"
```

Microsoft SQL:

```python
def select_format(self):
    return "SELECT {limit} {columns} FROM {table}"
```

Simply changing the order in this string will allow us to replace the format of the SQL statement generated. The last step is to change exactly what the word is.

Again, MySQL is `LIMIT X` and Microsoft is `TOP X`. We can accomplish this by doing this.

MySQL:

```python
def limit_string(self):
  return "LIMIT {limit}"
```

and Microsoft:

```python
def limit_string(self):
  return "TOP {limit}"
```

Now we have abstracted the differences into their own classes. Now when we compile the string, everything falls into place:

```python
self.select_format().format(
    columns="*",
    table="`users`",
      limit="1"
)
```

Let's remove the abstractions a bit so we can see more low level what it would be doing:

MySQL:

```python
"SELECT {columns} FROM {table} {limit}".format(
    columns="*",
    table="`users`",
      limit="LIMIT 1"
)
#== 'SELECT * FROM `users` LIMIT 1'
```

Microsoft:

```python
"SELECT {limit} {columns} FROM {table} ".format(
    columns="*",
    table="`users`",
      limit="TOP 1"
)
#== 'SELECT TOP 1 * FROM `users`'
```

So notice here the abstractions can be changed per each grammar for databases with different SQL structures.

### Format Strings

The child grammar classes have a whole bunch of these statements for getting the smaller things like a table.

Most methods in the child grammar classes are actually just these strings.

MySQL tables are in the format of this:

```text
`users`
```

and Microsoft are this:

```text
[users]
```

So again we have the exact same thing on the grammar class like this:

```text
table = self.table_string().format(table=table)
```

Which unabstracted looks like this for MySQL:

```python
table = "`{table}`".format(table=table)
```

and this for Microsoft:

```python
table = "[{table}]".format(table=table)
```

There are a whole bunch of these. Column statements are the exact same thing.

## Compiling Methods

There are a whole bunch of methods that begin with `_compile` so let's explain what those are.

Now that all the differences between grammars are abstracted into the child grammar classes, all the heavy listing can be done in the `BaseGrammar` class which is the parent grammar class.

This `BaseGrammar` class is responsible for doing the actual compiling in the above section. So this class really just has a bunch of classes like `_compile_wheres`, `_compile_selects` etc.

The heart of this class really lies in the `_compile_select`, `_compile_create` methods. Most other `_compile` methods are really just to do the string interpolation explained above.

Take a look at what the abstracted format for compiling the query statement above.

Let's bring back the unabstracted version first:

```python
"SELECT {columns} FROM {table} {limit}".format(
    columns="*",
    table="`users`",
      limit="LIMIT 1"
)
#== 'SELECT * FROM `users` LIMIT 1'
```

And now what that method would really look like:

```python
"SELECT {columns} FROM {table} {wheres} {limit}".format(
    columns=self._compile_columns(),
    table=self._compile_from(),
      limit=self._compile_limit()
    wheres=self._compile_wheres
)
#== 'SELECT * FROM `users` LIMIT 1'
```

So notice we have a whole bunch of `_compile` methods but they are mainly just for compiling the select, create or alter statements.

## Models and Query Builder

Models and query builders are really hand in hand. In almost all cases, a single method on the model will pass everything off to the `QueryBuilder` class.

Just know the Model is really just a small proxy for the `QueryBuilder`. Most methods on the model simply call the `QueryBuilder` so we will focus on the `QueryBuilder`.

The only thing the model class does is contains some small settings like the table name, the attributes after a database call is made \(query results\) and some other small settings like the connection and grammar to use.

It is important though to know the differences between class \(`cls`\) and an object instance. Be sure to read the section below.

### CLS

Since we instantiate the model later, we are typically working with a `cls` variable. This `cls` variable is the class of the model and not the instance of the model. You may want to research what the difference between classes and instances are if you don't know but one important thing to know is that:

> All attributes set on a class are distributed down to all instances.

This means that we can't do things like set the query result on the model class because it would defeat the purpose. Each model needs to have its own result.

The real reason we do this is really only for the visual effect. I would like to do this:

```python
User.find(1)
```

and not:

```python
User().find(1)
```

The second way would make things a bit easier but I am willing to go through a little more trouble to make sure the ORM looks really clean.

So just keep this in mind when working with models and the query builder

### Query Builder

This `QueryBuilder` class is responsible for building up the query so it will have a whole bunch of attributes on it that will eventually be passed off to the grammar class and compiled to SQL. That SQL will then be passed to the connection class and will do the database call to return the result.

This is really the meat and potatoes of the ORM and really needs to be perfect and will have the most features and will take the most time to build out and get right.

When you call `where` on the model it will pass the info to the query builder and return this `QueryBuilder` class.

```text
user = User.where('age', 18)
#== <masonite.orm.QueryBuilder object>
```

All additional calls will be done on the query builder object:

```python
user = User.where('age', 18).where('name', 'Joe').limit(1)
#== <masonite.orm.QueryBuilder object>
```

Finally when you call a method like `.get()` it will return a collection of results.

```python
user = User.where('age', 18).where('name', 'Joe').limit(1).get()
#== <masonite.orm.Collection object>
```

If you call `first()` it will return a single model:

```python
user = User.where('age', 18).where('name', 'Joe').limit(1).first()
#== <app.User object>
```

So again we use the `QueryBuilder` to build up a query and then later execute it.

### Expression Classes

There are a few different classes which will aid in the compiling of SQL from the grammar class. These really are just various classes with different attributes on them. These classes are:

* `QueryExpression` - Used for compiling of where statements
* `HavingExpression` - Used for the compiling of Having statements
* `JoinExpression` - Used for the compiling of Join statements
* `UpdateExpression` - Used for the compiling of Update statements.
* `SubSelectExpression` - Used for compiling sub selects. Sub selects can be placed inside where statements to make complex where statements more powerful
* `SubGroupExpression`- Used to be passed into a callable to be executed on later. This is useful again for sub selects but just a layer of abstraction for callables

These are simply used when building up different parts of a query. When the `_compile_wheres`, `_compile_update` and other methods are ran on the grammar class, these just make it more simple to fetch the needed data and are not too generic to make difficult use cases challenging to code for.

### QueryBuilder -&gt; Grammar

To be more clear, once we are done building the query and then call `.get()` or `.first()`, all the wheres, selects, group\_by's etc are passed off to the correct grammar class like `MySQLGrammar` which will then compile down to a SQL string.

### QueryBuilder -&gt; Connection

That SQL string is then sent to the connection class along with the bindings from the grammar class. We then have a result in the form of a dictionary. We don't want to be working with a bunch of dictionaries though, we want to work with more models.

### QueryBuilder Hydrating

The `QueryBuilder` object when returning the response is also responsible for hydrating your models. Hydrating is really just a fancy word for filling dummy models with data. We really don't want to work with dictionaries in our project so we take the dictionary response and shove it into a Model and return the model. Now we have a class much more useful than a simple dictionary.

For times we have several results \(a list of dictionaries\) we simply loop through the list and fill a different model with each dictionary. So if we have a result of 5 results we loop through each one and build up a collection of 5 hydrated models. We do this by calling the `.hydrate()` method which creates a new instance and hydrates the instance with the dictionary.

## Relationships

Relationships are a bit magical. We needed to do some Python class management magic to get nail the inherently magical nature of the relationship classes. For example we have a relationship like this:

```python
class User:

        @belongs_to('local_key', 'foreign_key')
    def profile(self):
        return Profile
```

This is innocent enough but we would like when you access something like this:

```python
user = User.find(1)
user.profile.city
```

to work. Very strange I know. How would we get an attribute accession to:

* find the correct model in the method
* build the query
* Find the correct foreign key's to fetch on
* return a fully hydrated model ready to go

For this we do some decorator and attribute accession magic using the `__get__` magic method which is called whenever an attribute is accessed. We can then hijack this hook and return whatever we need. In this case, a fully hydrated model.

### Relationship classes

Its useful to explain the relationship classes.

We have a `BaseRelationship` class which really just contains all the magic we need for the actual decorator to work.

We then have a `BelongsTo` relationship \(which is imported `as belongs_to` in the `__init__.py` file so this is where the name change comes from in the decorator\) which has a simple `apply_query` method with does the query needed to return the connection using the models `QueryBuilder`. Here we have `foreign` and `owner` variables. `foreign` is the relationship class \(In this case, `Profile`\) and `owner` is the current model \(in this case `User`\).

The query is applied and returns a result from the query builder in the form of a dictionary or a list \(for one result it will be a dictionary and if multiple are returned it will be a list\). Then the normal process takes its course. If a dictionary it will return a hydrated model and if a list is returned it will return a collection of hydrated models.

## Schema Class

The Schema class is responsible for the creation and altering of tables so will have a slightly different syntax for building a normal Query Builder class. Here we don't have things like `where` and `limit`. Instead of have things in the format of:

```text
CREATE TABLE `table` (
    `name` VARCHAR(255)
)
```

So the format is a bit different but we can do the exact same kind of string interpolation as above. I won't go over how to do that again but know we again have something like this:

```python
def create_start(self):
    return "CREATE TABLE {table}"
```

That part right there may be abstracted again into using the full statement like the select but for now its broken up a bit more.

### Classes

So now let's talk about how each class talks to eachother here.

### Schema -&gt; Blueprint

The Schema class is responsible for specifying the table and/or the connection to use. It will then will pass that information off to the `Blueprint` class which really is the same thing as the relationship between `Model` and `QueryBuilder`. The Schema class is also responsible for setting either the `create` or `alter` modes. This is set if you either use `Schema.create('users')` or `Schema.table('users')` respectively.

The `Blueprint` class is similiar to the `QueryBuilder` class because both simply build up a bunch of columns to to act on. One is just used for fetching data and the other is used for changing or creating tables.

The Schema class calls the blueprint class as a context manager.

The blueprint class will be built up in this format:

```python
Schema.table('users') as blueprint:
    blueprint.string('name')
  blueprint.integer('age')
```

Notice we are just building up a blueprint class.

### Blueprint -&gt; Column

The blueprint class passes the information given to it and builds up a list of columns using the `Column` class. These are stored in an attribute of a tuple of columns to later be passed and compiled to SQL by the grammar class.

```text
blueprint._columns
#== (<masonite.orm.Column object>, <masonite.orm.Column object>,)
```

Then the blueprint class is either set to a create or an Alter by the Schema class.

### Compiling

Finally we need to compile the query which is simply done by doing `blueprint.to_sql()` which will either build a `create` or `alter` query depending on what was originally set by the `Schema` class before.

