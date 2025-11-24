---
title: "PostgreSQL database"
date: 2025-08-13T19:30:08+10:00
draft: false
group: true
weight: 200
summary: "How to work with Postgres databases."
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


### Opening a database

What's new: `open` `postgres-schema`

To open a connection to a PostgreSQL database server, we use generic function `Open` function with a PostgreSQL connection URI.

```lisp
db: Open postgres://username:password@localhost:5432/database_name
```

> **Open** is a generic function that dispatches on the kind of first argument. In this case the argument is of kind postgres-schema.

### Simple query

To query a PostgreSQL database we use the `query` function. Let's start with a simple query to get the current timestamp from PostgreSQL.

```lisp  
db .Query { select current_timestamp }
; returns a table, so we display it
|display
; 
; | current_timestamp              |
; +--------------------------------+
; | 2025-01-08 14:30:45.123456+00  |
```

Let's make another query to demonstrate PostgreSQL's capabilities.

```lisp
db .Query { select current_date as today , 'Hello psql' as message , 42 * 2 as answer }
; returns a table, which we again ...
|display
;
; | today      | message    | answer |
; +----------------------------------|
; | 2025-01-08 | Hello psql | 84     |
```

> **Query** is a generic function that dispatches on the kind of first argument. In this case a PostgreSQL database connection.

### Executing SQL

SQL that doesn't query for data, but executes changes is called with an `exec` function. We will first use this to create a table.

```lisp
db .Exec { 
    create table employees (  
        id serial primary key , 
        name varchar ( 100 ) , salary decimal ( 10 , 2 ) , 
        hire_date date default current_date 
    ) 
}
```

We can use function `exec` also to insert some data to our table.

```lisp
db .Exec { 
    insert into employees   (name, department, salary) 
    values ( 'Alice Johnson' , 'Engineering' , 75000.00 ) 
}

; PostgreSQL also allows us to insert multiple rows at once
db .Exec { insert into employees ( name , salary ) values 
              ( "Bob Smith" , 60000.00 ) ,
              ( "Carol Davis" , 80000.00 ) , 
              ( "David Wilson" , 55000.00 ) }
```

Function `Exec` returns an integer (1 for success) or a failure.

### Querying

We now have a database with a table and some data in it. So we can do what we usually do with databases - we `Query` them.

```lisp
x> .Query { select * from employees } |display
; | id | name          | salary   | hire_date            |
; +------------------------------------------------------+
; | 1  | Alice Johnson | 75000.00 | 2025-09-26T00:00:00Z |
; | 2  | Bob Smith     | 60000.00 | 2025-09-27T00:00:00Z |
; | 3  | Carol Davis   | 80000.00 | 2025-09-27T00:00:00Z |
; | 4  | David Wilson  | 55000.00 | 2025-09-27T00:00:00Z |
```

Function `Query` returns a table or a failure value type.

### Working with PostgreSQL-specific features

PostgreSQL offers many advanced features. Here are some examples:

#### JSON operations

```lisp
; Create a table with JSON data
db .Exec { create table products ( 
    id serial primary key ,
    name varchar ( 100 ) ,
    metadata jsonb 
) }

db .Exec { insert into products ( name , metadata ) values 
    ( `Laptop` , `{"brand": "Dell", "specs": {"ram": "16GB", "cpu": "Intel i7"}}` ) ,
    ( `Phone` , `{"brand": "Apple", "specs": {"storage": "256GB", "color": "black"}}` ) }

; Query JSON data
db .Query `select name, metadata->>'brand' as brand, 
            metadata->'specs'->>'ram' as ram 
            from products 
            where metadata->>'brand' = 'Dell'`
; | name   | brand | ram |
; +----------------------+
; | Laptop | Dell  | 16GB|
```

#### Arrays

```lisp
; text[] is not a walid Rye word for now, so string
db .Exec `create table teams ( 
    id serial primary key ,
    name varchar ( 100 ) ,
    members text[]
)`

db .Exec { insert into teams ( name , members ) values 
    ( "Backend Team" , array [ "Alice" , "Bob" , "Charlie" ] ) ,
    ( "Frontend Team" , array [ "Diana" , "Eve" , "Frank" ] ) }

db .Query { select name , array_length ( members , 1 ) as team_size 
            from teams 
            where "Alice" = any ( members ) }
; | name         | team_size |
; +--------------------------+
; | Backend Team | 3         |
```

### Prepared statements with get-words

Like with SQLite, Rye's SQL dialect allows us to embed values directly using get-words, which translates to safe prepared statements.

```lisp
; Define some variables
min-salary: 60000

; Use get-words (:variable-name) to safely embed values
db .Query { select 
             name , salary 
             from employees 
            where salary >= ?min-salary }
|display
; | name          | salary   |
; +--------------------------+
; | Alice Johnson | 75000.00 |
; | Bob Smith     | 60000.00 |
; | Carol Davis   | 80000.00 |

```

This approach provides the security benefits of prepared statements while maintaining readable SQL code.

### Transactions

PostgreSQL connections should be properly managed in production applications:

```lisp
; For applications with multiple database operations,
; reuse the same connection object
perform-batch-operations: fn { db } {
    db .Exec { begin }
    
    ; Multiple operations here
    db .Exec { insert into employees ( name ) values ( "New Employee" ) }
    db .Exec { update employees set salary = salary * 1.05 }
    
    db .Exec { commit }
}

db |perform-batch-operations
```

<!--

### Advanced PostgreSQL features

PostgreSQL supports many advanced features that you can use through Rye:

#### Window functions

```rye
db .query { select name, salary, department,
            rank() over (partition by department order by salary desc) as dept_rank,
            avg(salary) over (partition by department) as dept_avg
            from employees }
```

#### Common Table Expressions (CTEs)

```rye
db .query { with dept_stats as (
                select department, 
                       count(*) as emp_count,
                       avg(salary) as avg_salary
                from employees 
                group by department
            )
            select * from dept_stats 
            where emp_count > 1 }
```

#### Full-text search

```rye
; Create a table with full-text search capability
db .Exec { create table articles (
    id serial primary key,
    title text,
    content text,
    search_vector tsvector
) }

db .Exec { create index on articles using gin(search_vector) }
```

### Connection strings

PostgreSQL connection strings support various formats and options:

```rye
; Basic connection
db1: open postgres://user:password@localhost:5432/mydb

; Connection with SSL
db2: open postgres://user:password@localhost:5432/mydb?sslmode=require

; Connection with custom port and options
db3: open postgres://user:password@localhost:5433/mydb?connect_timeout=10&application_name=rye_app
```

### Error handling

PostgreSQL operations can fail, and Rye provides ways to handle these gracefully:

```rye
result: db .query { select * from non_existent_table }
|if failure {
    "Query failed: " + .msg
} {
    "Query succeeded"
    result |display
}
```

This PostgreSQL integration in Rye provides a powerful way to work with one of the world's most advanced open-source databases, while maintaining Rye's philosophy of clear, readable code.

-->
