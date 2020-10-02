# White Page

## Outline ORM White Paper

> **This project is not currently in Masonite but is in development. This article is designed to be a learning document to aid in those who wish to contribute to the project. This article will be updated when concepts are added to the ORM project or when concepts change. You can contribute to the project at the** [**Masonite ORM Repo**](https://github.com/MasoniteFramework/orm)

## The Flow

I will discuss the flow at a high level first and then can talk about each part separately.

There are a few different paths you can start out with. Not everything starts out at the model level. You may use the query builder class directly to build your queries. The query builder class is exactly that: a class to build queries. So you will interact with this class (more on the class later) and it will set things like wheres, limits, selects, etc to the class and then pass all that off to build a query.

### The Model

First let's talk about the flow of the `Model`. The `Model` is probably what most people will be using the majority of the time. The `Model` is basically an wrapper entity around a table. So 1 table will likely equal to 1 model. A `users` table will have a `User` model and a `articles` table will have an `Article` model. 

The interesting things about the `Model` is that its just a shell around the `QueryBuilder` class. The majority of the time you call something on the `Model` it's actually just building a query builder class immediately and passing the rest of the call off. This is important to understand:

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

Finally when we are done building a query we will call a `.get()` which is basically an executionary command:

```python
user.select('id').where('id', 1).where('active', 1).get() #== <masonite.orm.Collection object>
```

When you call `get`, the query builder will pass everything you built up \(1 select, 2 where statements\) and pass those into a `Grammar` class. The `Grammar` class is responsible for looping through the 3 statements and compiling them into a SQL query that will run. So the `Grammar` class will compile a query that looks like this:

```text
SELECT `id` FROM `users` WHERE `id` = '1' AND `active` = 1
```

If it needs to build a Qmark query (a query with question marks which will be replaced with query bindings to prevent SQL injection) then it will look like this:

```text
SELECT `id` FROM `users` WHERE `id` = '?' AND `active` = ?
```

and have 2 query bindings:

```python
(1,1)
```

Once we get the query we can then pass the query into the connection class which will connect to the MySQL database to send the query.

We will then get back a dictionary from the query and "hydrate" the original model. When you hydrate a model it simply means we set the dictionary result into the model so when we access something like `user.name` we will get the name of the user. Think of it as loading the dictionary into the class to be used later during accession and setting.

## Grammar Classes

Grammar classes are classes which are responsible for the compiling of attributes into a SQL statement. The SQL statement will then be given back to whatever called it \(like the `QueryBuilder` class\) and then passed to the connection class to make the database call and return the result. Again the grammar class is only responsible for compiling the query into a string. Simply taking attributes passed to it and looping through them and compiling them into a query.

The grammar class will be responsible for both SQL and Qmark. Again, SQL looks like this:

```text
SELECT * FROM `users` where `age` = '18'
```

And Qmark is this:

```text
SELECT * FROM `users` where `age` = '?'
```

Qmark queries will then be passed off to the connection class with a tuple of bindings like `(18,)`. This helps protect against SQL injection attacks. **All queries passed to the connection class should be the qmark query. Compiling SQL is really for debugging purposes while developing. Passing straight SQL into the connection class could leave queries open to SQL injection.**

**Any values should be able to be qmarked**. This is done inside the grammar class by replacing the value with a `'?'` and then adding the value to the bindings. The grammar class knows it should be qmarked by passing the qmark boolean variable throughout the grammar class.

The grammar class is also really an abstraction as well. All the heavy lifting is done inside the `BaseGrammar` class. Child classes \(like `MySQLGrammar` and `PostgresGrammar`, etc\) really just contain the formatting of the sql strings.

**Currently there are 2 different grammar classes for each of the supported grammars. There is one for normal queries and one for schema queries. They could be 1 big class but the class would be giant and it is hard to maintain a god class like this responsable for everything. It also makes it harder to first build the grammar up for quering (selects, updates, deletes, etc) and then later support schema building.**

Almost all SQL is bascially the same but with slightly different formats or placements for where some syntax goes. This is why this structure we use is so powerful and easy to expand or fix later on.

For example, MySQL has this format for select statements with a limit:

```text
SELECT * from `users` LIMIT 1
```

But Microsoft SQL Server has this:

```text
SELECT TOP 1 * from `users`
```

Notice the SQL is bascially the same but the limiting statement is in a different spot in the SQL.

We can accomplish this by specifying the general: select, insert, update and delete formats so we can better organize and swap the placement later. We do this by using Python keyword string interpolation. For example let's break down to a more low level way on how we can accomplish this:

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

Again, MySQL is `LIMIT X` and Microsoft is `TOP X`. We can accomplish this by specifying the differences in their own method. Remember these are all in the subclasses of the grammar class. Mysql is in `MySQLGrammar` and Microsoft is in `MSSQLGrammar`

MySQL:

```python
# MySQLGrammar 

def limit_string(self):
  return "LIMIT {limit}"
```

and Microsoft:

```python
# MSSQLGrammar

def limit_string(self):
  return "TOP {limit}"
```

Now we have abstracted the differences into their own classes and class methods. Now when we compile the string, everything falls into place. This code snippet is located in the `BaseGrammar` class (which calls the supported grammar class we built above). 

```python
# Everything completely abstracted into it's own class and class methods.
sql = self.select_format().format(
    columns=self.process_columns(),
    table=self.process_table(),
    limit=self.process_limit()
)
```

Let's remove the abstractions and explode the variables a bit so we can see more low level what it would be doing:

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

So notice here the abstractions can be changed per each grammar for databases with different SQL structures. You just need to change the response of the string returning methods and the structure of the `select_format` methods

### Format Strings

The child grammar classes have a whole bunch of these statements for getting the smaller things like a table

Most methods in the child grammar classes are actually just these strings.

MySQL tables are in the format of this:

```text
`users`
```

Postgres and SQLite tables are in the format of this:

```text
"users"
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
# MySQL
table = "`{table}`".format(table=table)
```

and this for Microsoft:

```python
# MSSQL
table = "[{table}]".format(table=table)
```

There are a whole bunch of these methods in the grammar classes for a whole range of things. Any differences that there can possible be between databases are abstracted into these methods.

## Compiling Methods

There are a whole bunch of methods that begin with `process_` or `_compile_` so let's explain what those are.

Now that all the differences between grammars are abstracted into the child grammar classes, all the heavy listing can be done in the `BaseGrammar` class which is the parent grammar class and really the engine behind compiling the queries for all grammars.

This `BaseGrammar` class is responsible for doing the actual compiling in the above section. So this class really just has a bunch of classes like `process_wheres`, `process_selects` etc. These are more supporting methods that help process the sql strings for the `_compile_` methods.

There are also methods that begin with `_compile_`. These are for responsable for compiling the actual respective queries. The heart of this class really lies in the `_compile_select`, `_compile_create`, `_compile_update`, `_compile_delete` methods.

Let's bring back the unabstracted version first:

```python
def _compile_select(self):
    "SELECT {columns} FROM {table} {limit}".format(
        columns="*",
        table="`users`",
        limit="LIMIT 1"
    )
#== 'SELECT * FROM `users` LIMIT 1'
```

Now let's start abstracting until we get what is really in the class.

And now what that method would really look likes with the supporting `_compile` methods in place:

```python
def _compile_select(self):
    "SELECT {columns} FROM {table} {wheres} {limit}".format(
        columns=self.process_columns(),
        table=self.process_from(),
        limit=self.process_limit()
        wheres=self.process_wheres
    )

    #== 'SELECT * FROM `users` LIMIT 1'
```

So notice we have a whole bunch of `_compile` methods but they are mainly just for supporting the main compiling of the select, create or alter statements.

And now finally what the method actually looks like in the class:

```python
def _compile_select(self):
    self.select_format().format(
        columns=self.process_columns(),
        table=self.process_from(),
        limit=self.process_limit()
        wheres=self.process_wheres
    )
    #== 'SELECT * FROM `users` LIMIT 1'
```

## Models and Query Builder

Models and query builders are really hand in hand. In almost all cases, a single method on the model will pass everything off to the `QueryBuilder` class immediately.

Just know the Model is really just a small proxy for the `QueryBuilder`. Most methods on the model simply call the `QueryBuilder` so we will focus on the `QueryBuilder`.

The only thing the model class does is contains some small settings like the table name, the attributes after a database call is made \(query results\) and some other small settings like the connection and grammar to use.

It is important though to know the differences between class \(`cls`\) and an object instance. Be sure to read the section below.

### Meta Classing

One of the trickier bits of magic we have when it comes to the model is we set a meta class on the `Model` class (the base class that all of your `User` and `Article` models will inherit). What this does is essentially creates a middleware between first calling methods. Since its really hard to do everything while handling different class instantances and class classes it's easier to catch the call and turn it into an instance before moving on. 

This is hard to explain but let's see what this really solves:

We COULD just do this with everything:

```python
class User(Model):
    pass
```

And then perform model calls:

```python
result = User().where('...')
```

But it doesn't look as clean as:

```python
result = User.where('...')
```

(Also for backwards compatability with Orator it would be a huge change if we didn't support this).

So if you look at the `Model.py` class we have a meta class inherited (you'll notice if you look at the file) which actually does a bit of magic and actually instanitates the class before any methods are called. This is similiar to any normal Python hook you can tie into like `__getattr__`. 

**This makes handling `cls` and `self` much easier. Although there are special use cases where we need to handle cls directly which is why you will see some `@classmethod` decorators on some model methods.**

### Pass Through

We mentioned that the model simply constructs a query builder and essentially passes everything off to the query builder class. 

The issue though is that when you call something like `User.where(..)` it will call the where on the User class. Since theres actually no `where` method on the model class it will hook into the `__getattr__` on the model class. From there we catch a bunch of different methods located in the `__passthrough__` attribute on the model and pass that right off to the query builder. This is important to understand.

## Query Builder

This `QueryBuilder` class is responsible for building up the query so it will have a whole bunch of attributes on it that will eventually be passed off to the grammar class and compiled to SQL. That SQL will then be passed to the connection class and will do the database call to return the result.

The `QueryBuilder` class is really the meat and potatoes of the ORM and really needs to be perfect and will have the most features and will take the most time to build out and get right.

For example, when you call `where` on the model it will pass the info to the query builder and return this `QueryBuilder` class.

```python
user = User.where('age', 18)
#== <masonite.orm.QueryBuilder object>
```

All additional calls will be done on THAT query builder object:

```python
user = User.where('age', 18).where('name', 'Joe').limit(1)
#== <masonite.orm.QueryBuilder object x100>
```

Finally when you call a method like `.get()` it will return a collection of results.

```python
user = User.where('age', 18).where('name', 'Joe').limit(1).get()
#== <masonite.orm.Collection object x101>
```

If you call `first()` it will return a single model:

```python
user = User.where('age', 18).where('name', 'Joe').limit(1).first()
#== <app.User object x100>
```

So again we use the `QueryBuilder` to build up a query and then later execute it.

### Expression Classes

There are a few different classes which will aid in the compiling of SQL from the grammar class. These really are just various classes with different attributes on them. They are internal only classes made to better compile things inside the `BaseGrammar` class, since we use things like isinstance checks and attribute conditionals. You will not be using these directly when developing applications. These classes are:

* `QueryExpression` - Used for compiling of where statements
* `HavingExpression` - Used for the compiling of Having statements
* `JoinExpression` - Used for the compiling of Join statements
* `UpdateExpression` - Used for the compiling of Update statements.
* `SubSelectExpression` - Used for compiling sub selects. Sub selects can be placed inside where statements to make complex where statements more powerful
* `SubGroupExpression`- Used to be passed into a callable to be executed on later. This is useful again for sub selects but just a layer of abstraction for callables

These are simply used when building up different parts of a query. When the `_compile_wheres`, `_compile_update` and other methods are ran on the grammar class, these just make it more simple to fetch the needed data and are not too generic to make difficult use cases challenging to code for.

## How classes interact with eachother

### Model -&gt; QueryBuilder

The Model passes off anything set on it directly to the query builder once accessed. All calls after will be based on a new query builder class. All query building will be done on this class.

### QueryBuilder -&gt; Grammar

To be more clear, once we are done building the query and then call `.get()` or `.first()`, all the wheres, selects, group\_by's etc are passed off to the correct grammar class like `MySQLGrammar` which will then compile down to a SQL string.

### QueryBuilder -&gt; Connection

That SQL string returned from the grammar class is then sent to the connection class along with the bindings from the grammar class. We then have a result in the form of a dictionary. We don't want to be working with a bunch of dictionaries though, we want to work with more models.

### QueryBuilder Hydrating

The `QueryBuilder` object when returning the response is also responsible for hydrating your models if a model is passed in. If no model is passed into the initializer then it will just return a dictionary or list. Hydrating is really just a fancy word for filling dummy models with data. We really don't want to work with dictionaries in our project so we take the dictionary response and shove it into a Model and return the model. Now we have a class much more useful than a simple dictionary.

For times we have several results \(a list of dictionaries\) we simply loop through the list and fill a different model with each dictionary. So if we have a result of 5 results we loop through each one and build up a collection of 5 hydrated models. We do this by calling the `.hydrate()` method which creates a new instance and hydrates the instance with the dictionary.

## Relationships

**RELATIONSHIPS ARE STILL A WORK IN PROGRESS AND SUBJECT TO CHANGE**

Relationships are a bit magical and uses a lot of internal low level Python magic to get right. We needed to do some Python class management magic to nail the inherently magical nature of the relationship classes. For example we have a relationship like this:

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

BUT we also want to be able to extend the relationship as well:

```python
user = User.find(1)
user.profile().city
```

so we need to both access the attribute AND call the attribute. Very strange I know. How would we get an attribute accession to:

* find the correct model in the method
* build the query
* Find the correct foreign key's to fetch on
* return a fully hydrated model ready to go
* but when you call it simple do the wheres and return the query builder.

For this we do some decorator and attribute accession magic using the `__get__` magic method which is called whenever an attribute is accessed. We can then hijack this hook and return whatever we need. In this case, a fully hydrated model or a query builder.

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

