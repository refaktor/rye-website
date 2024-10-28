---
title: "Spreadsheets*"
weight: 50
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Functions for working with spreadsheet value type."
mygroup: true

---

*work-in-progress*


The idea goes like this ... humans naturally think about certain types of information in a list-like format. However, when it comes to multidimensional information, we don't conceptualize it as lists of maps or dictionaries. These are computer concepts, not human, or just information concepts.

Instead, we intuitively organize multidimensional information in a tabular format. This is a more natural and accessible way for humans to process and understand complex(er) data.

To create a "more" high-level programming language, we need higher-level value types. These higher-level types then in turn enable more expressive and intent-based methods, which elevates the entire vocabulary of the language or of way of thinking forward.

## Creating a spreadsheet

```clojure
spreadsheet { "name" "score" } { "Bae" 299 "Jane" 405 "Jim" 302 }
; creates a spreadsheet with two columns and 3 rows

spreadsheet\rows { "name" "score" }                ; todo
 { { "Bae" 299 } { "Jane" 405 } { "Jim" 302 } }
; creates a spreadsheet with two columns and 3 rows
```

## Loading a spreadsheet

### From CSV file

```clojure
load\csv %file.csv
; returns a spreadsheet
```

### From SQL query

```clojure
open sqlite://examples/cookbook/sample.s3db
|query { select * from players }
; returns a spreadsheet
```

### From JSON file

```clojure

```

## Generating a spreadsheet

### Generating random sample

How would you make a spreadsheet in code. For example we want a random spreadsheet of 5 players with
scores from 200 to 500.

```clojure
names: { "Jane" "Jim" "Bob" "Abbe" "Heather" "Sasha" }

produce 5 { } 
 { + [ random names  200 + random\integer 300 ] }
|spreadsheet* { "name" "score" }

; displays:
;
; | name | score |
; +--------------+
; | Jim  | 412   |
; | Jane | 273   |
; | Abbe | 346   |
; | Jim  | 221   |
; | Sasha| 275   |

```



## Introspecting a spreadsheet

```clojure
spreadsheet { "name" "score" } { "Bae" 299 "Jane" 405 "Jim" 302 } :spr

; get the number of rows
spr .length?

; get the column names
spr .header?

; get the types of columns

; display it
spr .display

```

## Spreadsheet for information processing

These functions are all referentially transparent and symmetric. They accept a spreadsheet and return a new spreadsheet. This means you
can transparently combine them together, to achieve what you want.

### Autotype

### Selecting columns

### Filtering

```clojure
```
### Adding calculated rows


### Grouping

### Joining spreadsheets

### Limiting (limit)


## Using spreadsheet for storage


```clojure
```

## Saving a spreadsheet
