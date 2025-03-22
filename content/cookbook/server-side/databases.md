---
title: "Databases*"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 100
summary: "How to work with various SQL databases."
mygroup: true
---

<style>
  pre > code {
      __width: 440px;
      border-radius: 5px;
      __padding: 16px;
      __background-color: #3e3e3e;
      font-size: 13px;
  }
</style>

*work in progress*

## SQLite 3

### Opening a database

What's new: `open` `sqlite-schema`

To open or create a local file as a sqlite database the `open` function.

```lisp
db: open sqlite://sample.s3db
```

> **open** is a generic function that dispatches on the kind of first argument. In this case
> the argument is of kind sqlite-schema.

### Simple query

To query a database we use a function query. Since we have no tables in the database we
just created we will just query for current date.

```lisp
db .query { select date() }
; returns a table, so we display it
|display
; 
; | date()       |
; +--------------+
; | 2025-01-07   |
```
&nbsp;

Let's make another query without actually requring and data in our database.

```lisp
db .query { select date() as today , 123 + 234 as nice_number }
; returns a table, which we again ...
|display
;
; | today      | nice_number |
; +--------------------------+
; | 2025-01-08 | 357         |
```

> you don't need to worrie about it, but **query** is again a generic function that dispatches on
> the kind of first argument. In this case a `sqlite-database`

### Executing SQL

SQL that doesn't query for data, but executes changes is called with a `exec` function. We will first use
this to create a table itself.


```lisp
db .exec { create table pets ( id int primary_key , name varchar(40) ) }
```
&nbsp;

We can use function `exec` also to insert some data to our table.

```lisp
db .exec { insert into pets values ( 1 , "Toto" ) }

; SQL also allows use to insert multiple rows at once
db .exec { insert into pets values ( 2 , "Hedwig" ) ,
                  ( 3 , "Nemo" ) , ( 4 , "Hooch"  ) }
```
&nbsp;

Function `exec` returns a database object.

### Querying

We finally have a database with a table and some data in it. So we can do what we usually do with databases, we 
`query` them.

```lisp
db .query { select * from pets }
|display
; 
; | id | name     |
; +---------------+
; | 1  | Toto     |
; | 2  | Hedwig   |
; | 3  | Nemo     |
; | 4  | Hooch    |
```
&nbsp;

Now let's query just the names starting with "H". 

```lisp
db .query { select * from pets where name like "H%" }
; 
; | id | name     |
; +---------------+
; | 2  | Hedwig   |
; | 4  | Hooch    |
```
&nbsp;

Function `query` returns a table or a failure value type.

## SQL dialect

The functions above, `execute` and `query` both accepted a database connection and something that looked like SQL.

But that SQL was not an string. It was a block of Rye values, not that different that Rye code is a block of Rye values. 

```python
# In Python, SQL is a string
res = db.execute("select name from pets")
res.fetchone()
```
&nbsp;


It turns out that Rye code is flexible enough to be able to represent SQL. There are some specifics, this is still Rye code, so rules of Rye code applies. For one, all tokens must be separated by spaces.

```lisp
; in Rye it's a block of code
db .query { select name from pets }
|first
```
&nbsp;


### Benefits?

In most programming language SQL is represented as a rather opaque string, and you use string operations to customize it. 


In Rye it is a block of Rye values, and sub blocks are actually sub-blocks. I think
this makes it better to work with as Rye tokens you work with are closer to SQL tokens and not just a flat string.



### Prepared statements

And there is a more concrete benefit. In case of SQL, we don't want to manually embed values into strings. This opens us to all sorts of problems and also security issues. That's why it's all but mandatory, to 
use prepared statements. But ordinary prepared statements have a weakness. The force us to rely on positional mapping. The "?" in a string have to be in the right order as the values we embed. And changing one or another
can quickly lead to bugs. Rye's SQL dialect let's us define values we want to embed directly in their places, but this translates to normal prepared statements, not undesired string embedding.

To do this we use the fact that Rye has many word types. To embed a value we use get-words, which also matches their normal behaviour.

## Postgres database


## MySQL database
