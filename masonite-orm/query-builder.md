# Query Builder

## Query Builder

## Preface

The query builder is a class which is used to build up a query for execution later. For example if you need multiple wheres for a query you can chain them together on this `QueryBuilder` class. The class is then modified until you want to execute the query. Models use the query builder under the hood to make all of those calls. Many model methods actually return an instance of `QueryBuilder` so you can continue to chain complex queries together.

Using the query builder class directly allows you to make database calls without needing to use a model.

## Getting the QueryBuilder class.

To get the query builder class you can simply import the query builder. Once imported you will need to pass the `connection_details` dictionary you store in your `config.database` file:

```python
from masoniteorm.query import QueryBuilder

builder = QueryBuilder().table("users")
```

You can then start making any number of database calls.

# Fetching Records

## Select

```python
builder.table('users').select('username').get()
# SELECT `username` from `users`
```

## First

You can easily get the first record:

```python
builder.table('users').first()
# SELECT `username` from `users` LIMIT 1
```

## All Records

You can also simply fetch all records from a table:

```python
builder.table('users').all()
# SELECT * from `users`
```

## The Get Method

Once you start chaining methods you should call the `get()` method instead of the `all()` method to execute the query.

For example, this is correct:

```python
builder.table('users').select('username').get()
```

And this is wrong:

```python
builder.table('users').select('username').all()
```

## Wheres

You may also specify any one of these where statements:

The simplest one is a "where equals" statement. This is a query to get where `username` equals `Joe` AND `age` equals `18`:

```python
builder.table('users').where('username', 'Joe').where('age', 18).get()
```

You can also specify comparison operators:

```python
builder.table('users').where('age', '=', 18).get()
builder.table('users').where('age', '>', 18).get()
builder.table('users').where('age', '<', 18).get()
builder.table('users').where('age', '>=', 18).get()
builder.table('users').where('age', '<=', 18).get()
```

## Where Null

Another common where clause is checking where a value is `NULL`:

```python
builder.table('users').where_null('admin').get()
```

This will fetch all records where the admin column is `NULL`.

Or the inverse:

```python
builder.table('users').where_not_null('admin').get()
```

This selects all columns where admin is `NOT NULL`.

## Where In

In order to fetch all records within a certain list we can pass in a list:

```python
builder.table('users').where_in('age', [18,21,25]).get()
```

This will fetch all records where the age is either `18`, `21` or `25`.

## Limits / Offsets

It's also very simple to use both limit and/or offset a query.

Here is an example of a limit:

```python
builder.table('users').limit(10).get()
```

Here is an example of an offset:

```python
builder.table('users').offset(10).get()
```

Or here is an example of using both:

```python
builder.table('users').limit(10).offset(10).get()
```

## Between

You may need to get all records where column values are between 2 values:

```python
builder.table('users').where_between('age', 18, 21).get()
```

## Group By

You may want to group by a specific column:

```python
builder.table('users').group_by('active').get()
```

## Having

Having clauses are typically used during a group by. For example, returning all users grouped by salary where the salary is greater than 0:

```python
builder.table('users').sum('salary').group_by('salary').having('salary').get()
```

You may also specify the same query but where the sum of the salary is greater than 50,000

```python
builder.table('users').sum('salary').group_by('salary').having('salary', 50000).get()
```

## Inner Joining

Joining is a way to take data from related tables and return it in 1 result set as well as filter anything out that doesn't have a relationship on the joining tables.

```python
builder.table('users').join('table1', 'table2.id', '=', 'table1.table_id')
```

This join will create an inner join.

You can also choose a left join:

## Left Join

```python
builder.table('users').left_join('table1', 'table2.id', '=', 'table1.table_id')
```

and a right join:

## Right Join

```python
builder.table('users').right_join('table1', 'table2.id', '=', 'table1.table_id')
```

## Increment

There are times where you really just need to increment a column and don't need to pull any additional information. A lot of the incrementing logic is hidden away:

```python
builder.table('users').increment('status')
```

Decrementing is also similiar:

## Decrement

```python
builder.table('users').decrement('status')
```

# Aggregates

There are several aggregating methods you can use to aggregate columns:

## Sum

```python
builder.table('users').sum('salary').get()
```

## Average

```python
builder.table('users').avg('salary').get()
```

## Count

```python
builder.table('users').count('salary').get()
```

## Max

```python
builder.table('users').max('salary').get()
```

## Min

```python
builder.table('users').min('salary').get()
```

# Raw Queries

If some queries would be easier written raw you can easily do so for both selects and wheres:

```python
builder.table('users').select_raw("COUNT(`username`) as username").where_raw("`username` = 'Joe'").get()
```

# Chunking

If you need to loop over a lot of results then consider chunking. A chunk will only pull in the specified number of records:

```python
for users in builder.table('users').chunk(100):
    for user in users:
        user #== <User object>
```

# Getting SQL

If you want to find out the SQL that will run when the command is executed. You can use `to_sql()`. This method returns the full query and is not the query that gets sent to the database. The query sent to the database is a "qmark query". This `to_sql()` method is mainly for debugging purposes.

See the section below for more information on qmark queries.

```python
builder.table('users').count('salary').to_sql()
#== SELECT COUNT(`users`.`salary`) FROM `users`
```

# Getting Qmark

Qmark is essentially just a normal SQL statement except the query is replaced with question marks. The values that should have been in the position of the question marks are stored in a tuple and sent along with the qmark query to help in sql injection. The qmark query is the actual query sent using the connection class.

```python
builder.table('users').count('salary').where('age', 18).to_sql()
#== SELECT COUNT(`users`.`salary`) FROM `users` WHERE `users`.`age` = '?'
```

# Updates

## Updating Records

You can update many records.

```python
builder.where('active', 0).update({
    'active': 1
})
# UPDATE `users` SET `users`.`active` = 1 where `users`.`active` = 0
```

You may update records as well.

# Deletes

## Deleting Records

You can delete many records as well. For example, deleting all records where active is set to 0.

```python
builder.where('active', 0).delete()
```

# Available Methods

* where
* where\_has
* first
* update
* create
* set\_scope
* set\_global\_scope
* select
* select\_raw
* create
* delete
* where
* where\_raw
* or\_where\[str, int, callable\]\)
* where\_exists"QueryBuilder"\]\):
* having
* where\_null
* where\_not\_null
* between
* not\_between
* where\_in
* where\_not\_in
* join
* left\_join
* right\_join
* where\_column
* limit
* offset
* update
* increment
* decrement
* sum
* count
* max
* order\_by
* group\_by
* aggregate
* first
* all
* get
* get\_grammar
* to\_sql
* to\_qmark

