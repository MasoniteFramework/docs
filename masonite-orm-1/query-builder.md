# 🧩 Query Builder

# Preface

The query builder is a class which is used to build up a query for execution later. For example if you need multiple wheres for a query you can chain them together on this `QueryBuilder` class. The class is then modified until you want to execute the query. Models use the query builder under the hood to make all of those calls. Many model methods actually return an instance of `QueryBuilder` so you can continue to chain complex queries together.

# Getting the `QueryBuilder` from a model

## Selects

## Fetching Records

* First
* all()
* get()

## Wheres

### Where Null

### Where In

## Raw Queries

## Limits / Offsets

## Where Has

## Updates

## Deletes

## Group By

## Having

## Joining

### Left

### Right

## Between

## Increment / Decrement

## Aggregates

* sum
* average
* count
* max
* min

## Getting SQL

* sql
* qmark


# Available Methods

- where
- where_has
- first
- update
- create
- set_scope
- set_global_scope
- select
- select_raw
- create
- delete
- where
- where_raw
- or_where[str, int, callable])
- where_exists"QueryBuilder"]):
- having
- where_null
- where_not_null
- between
- not_between
- where_in
- where_not_in
- join
- left_join
- right_join
- where_column
- limit
- offset
- update
- increment
- decrement
- sum
- count
- max
- order_by
- group_by
- aggregate
- first
- all
- get
- get_grammar
- to_sql
- to_qmark
- new

