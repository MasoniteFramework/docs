# Database Seeding

## Introduction

You'll likely want to seed your database during development in order to get some dummy data into your database to start working fast.

## Getting Started

We can simply create a new seeder by running:

```text
craft seed User
```

This will create a new seeder inside the `databases/seeds` directory. This will also create a `database_seeder.py` file which will be where the root of all seeds should be. 

The `user_table_seeder` should be where you simply abstract your seeds to.

## Running Seeds

We can run:

```text
craft seed:run
```

This will run the `database_seeder` seed. We can also specify an individual seed

```text
craft seed:run User
```

Which will run the `user_table_seeder` only.

{% hint style="success" %}
Read more about creating seeds with the [Orator documentation](https://orator-orm.com/docs/0.9/seeding.html).
{% endhint %}



