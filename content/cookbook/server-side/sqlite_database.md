---
title: "SQLite Database"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 100
summary: "How to work with SQLite databases in Rye."
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

## Opening a database

SQLite databases are file-based and easy to work with. Use the `open` function with a sqlite:// URI to create or connect to a database.

What's new: `Open` `sqlite-schema`

```lisp
rye .needs { sqlite }

db: Open sqlite://sample.s3db
```

The `open` function dispatches on the URI scheme type. For SQLite, it returns a database connection that you can use for queries and operations.

## Simple queries

Start with basic queries to test your connection. SQLite has built-in functions you can use without needing tables.

What's new: `Query` `display`

```lisp
; Test connection with current date
db .Query { select date() }
|display
; 
; | date()       |
; +--------------+
; | 2025-01-07   |

; Query multiple values and expressions
db .Query { select date() as today , 123 + 234 as sum , "Hello" as greeting }
|display
;
; | today      | sum | greeting |
; +-----------------------------+
; | 2025-01-08 | 357 | Hello    |
```

The `Query` function returns a table object that can be displayed or processed further.

## Creating tables and inserting data

Use `Exec` for SQL statements that modify the database structure or data but don't return results.

What's new: `Exec`

```lisp
; Create a table
db .Exec { create table pets ( id integer primary key , name varchar(40) , species varchar(20) ) }

; Insert single record
db .Exec { insert into pets values ( 1 , "Toto" , "dog" ) }

; Insert multiple records at once
db .Exec { insert into pets values ( 2 , "Hedwig" , "owl" ) ,
                                   ( 3 , "Nemo" , "fish" ) , 
                                   ( 4 , "Hooch" , "dog" ) }
```

The `Exec` function executes SQL statements and returns the database connection, allowing for method chaining.

## Querying data

Once you have data, you can query it with various conditions and operations.

```lisp
; Select all records
db .Query { select * from pets }
|display
; 
; | id | name   | species |
; +-----------------------+
; | 1  | Toto   | dog     |
; | 2  | Hedwig | owl     |
; | 3  | Nemo   | fish    |
; | 4  | Hooch  | dog     |

; Filter with WHERE clause
db .Query { select * from pets where species = "dog" }
|display

; Use LIKE for pattern matching
db .Query { select name from pets where name like "H%" }
|display
```

Queries return table objects that can be further processed with Rye's table functions.

## Prepared statements with parameters

Rye's SQL dialect supports safe parameter embedding using get-words. This prevents SQL injection and makes queries more readable.

What's new: get-words in SQL blocks

```lisp
; Set variables for parameters
pet-name: "Fluffy"
pet-species: "cat"
min-id: 2

; Use get-words (?variable) for parameters
db .Exec { insert into pets values ( null , ?pet-name , ?pet-species ) }

; Parameters work in queries too
db .Query { select * from pets where id > ?min-id and species = ?pet-species }
|display
```

The `?variable` syntax automatically creates prepared statements with proper parameter binding, making your code both safe and readable.

## Debugging SQL

Use `Show-SQL` to see the generated SQL and parameter values without executing the query.

What's new: `Show-SQL`

```lisp
search-term: "dog"
min-id: 1

; See what SQL is generated
db .Show-SQL { select * from pets where species = ?search-term and id >= ?min-id }
; Returns: "SELECT * FROM pets WHERE species = ? AND id >= ? 
;           -- Parameters: ? = dog, ? = 1"
```

This is useful for debugging complex queries and understanding how parameters are bound.

<!-- ## Working with results

Tables returned from queries can be processed with Rye's table functions and converted to other formats.

What's new: `htmlize` `first` `to-list`

```lisp
; Get query results
results: db .query { select * from pets where species = "dog" }

; Get first row
first-dog: first results
print first-dog

; Convert table to HTML
html-table: htmlize results
print html-table

; Process each row
results |each row { 
    print [ "Pet:" get row "name" "is a" get row "species" ]
}
```

The `htmlize` function is particularly useful for web applications where you need to display database results as HTML tables.

## Practical example: Simple blog

Here's a complete example showing how to build a simple blog system with SQLite.

```lisp
rye .needs { sqlite }

; Open database
blog-db: open sqlite://blog.db

; Create posts table
blog-db .exec { 
    create table if not exists posts ( 
        id integer primary key autoincrement ,
        title varchar(200) ,
        content text ,
        author varchar(100) ,
        created_date datetime default current_timestamp 
    ) 
}

; Insert sample posts
blog-db .exec { insert into posts ( title , content , author ) values 
    ( "Hello World" , "This is my first blog post!" , "Alice" ) ,
    ( "Rye Language" , "Learning about this amazing language." , "Bob" ) }

; Query recent posts
recent-posts: blog-db .query { 
    select id , title , author , created_date 
    from posts 
    order by created_date desc 
    limit 5 
}

; Display results
recent-posts |display

; Search for posts by author
author: "Alice"
author-posts: blog-db .query { 
    select title , content 
    from posts 
    where author = ?author 
}

author-posts |each post {
    print [ "Title:" get post "title" ]
    print [ "Content:" get post "content" ]
    print "---"
}
```

This example demonstrates table creation, data insertion, querying with parameters, and result processing in a real-world scenario.
-->