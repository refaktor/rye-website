---
title: 'SQLite and Postgres' 
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 400
summary: ""
mygroup: true
---

SQL dialect is a dialect of Rye values (not string) tha generates prepared SQL
statements withouth the mistake-prone need for positional arguments.

## SQLite

```rye
title: "some title"
content: "... content ..."

db: open sqlite://file.db
exec db { insert into notes ( title , content ) values ( ?title , ?content ) }

query db { select * from notes } |print
// prints:
// |title     |content        |
// |some title|... content ...|
```

## PostgreSQL

```rye
id: 101
db: open postgres://user1:password@demo
generate-token 32 :tok
exec db
{ insert into tokens ( id_user , token ) values ( ?id , ?tok )
  on conflict ( id_user ) do update set token = ?tok }
```
