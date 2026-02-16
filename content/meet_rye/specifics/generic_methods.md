---
title: Methods (generic functions)
date: 2025-02-11T19:30:08+10:00
draft: false
weight: 1800
summary: "Methods are functions that are dispatched based on Kind of first argument."
mygroup: true
---

Generic methods dispatch on the **Kind** of their first argument. By convention, they are capitalized. This enables Rye to use short, readable function names that work with different types of data.

## URIs and Kinds

URI is a syntax type recognized by the parser. Its prefix determines its **Kind**:

```clojure
https://ryelang.org |kind?                   ; returns: https-uri
sqlite://main.db    |kind?                   ; returns: sqlite-uri
postgres://user:pwd@localhost/db |kind?      ; returns: postgres-uri

; %readme.md is short for file://readme.md 
%readme.md |kind?                            ; returns: file-uri
```

## Generic Methods

Generic methods dispatch on the Kind of their first argument. This allows us to use short, meaningful words because the Kind provides the context:

```clojure
; These are two different functions defined in different modules
Open sqlite://main.db                ; opens a SQLite database
Open %main.md                        ; opens a file

; Same pattern with other generic words
Get https://ryelang.org              ; makes an HTTP GET request
Post https://ryelang.org "data"      ; makes an HTTP POST request
Write %world.txt "Hello!"            ; writes to a file

Read %config.txt                     ; reads from a file
Read https://example.com/data.txt    ; reads from URL
```

## Natives and Kinds

Native (Go) values that aren't represented by Rye value types are of type **Native**. Database connections, UI library objects, HTTP requests, file handles, etc. are all Natives.

Generic methods often return Natives with specific Kinds:

```clojure
; Open a database - returns a Native of kind sqlite-conn
Open sqlite://my.db |probe           
; prints: [Native of kind sqlite-conn]

; Create and open a file
Write %hello.txt "World!"
Open %hello.txt :f |probe               
; prints: [Native of kind file] 
```

## Listing Generic Methods

In the console, use `lg` to list generic methods available for a value:

```clojure
Open sqlite://my.db |lg
; Prints:
; Methods (sqlite-conn):
;  Exec [Builtin(2): Executes SQL that doesn't return rows]
;  Query [Builtin(2): Executes SQL query and returns results as a table]
;  Show-SQL [Builtin(2): Generates SQL string without executing]

Open %hello.txt |lg
; Prints:
; Methods (file):
;  Close [Builtin(1): Closes an open file]
;  Copy [Builtin(2): Copies content from a file to a writer]
;  Read-all [Builtin(1): Reads entire content as a string]
;  Reader [Builtin(1): Creates a new reader from file object]
;  ...
```

## Using Generic Methods

Once you have a Native, you can call its generic methods:

```clojure
; Working with a file
Write %hello.txt "World!"
f: Open %hello.txt 
f |Read-all |print                   ; prints: World!
f .Close

; Working with a database
db: Open sqlite://users.db
db .Query "SELECT * FROM users" |print
db .Close
```

## Why Generic Methods?

Generic methods provide several benefits:

1. **Readable code**: Use simple words like `Open`, `Read`, `Write`, `Close` instead of `file-open`, `sqlite-open`, etc.

2. **Consistent patterns**: The same word works across different types - `Open` opens files, databases, network connections, etc.

3. **Type safety**: The Kind ensures the right implementation is called - you can't accidentally call file methods on a database connection.

4. **Discoverability**: Use `lg` in the console to explore what methods are available for any value.

## Open Questions

There are still some design questions being explored:

* How to best document all available generic methods
* Whether more values should use Kinds
* How to handle method conflicts between Kinds
