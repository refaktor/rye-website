---
title: "Tables 2/3*"
weight: 52
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Functions for working with table value type."
mygroup: true
toc: true
---


## Endomorphic functions #2

### Adding a column

We can generate values for a new column from existing columns. This again creates a new table
with additional column.

```lisp
spr .add-column "Main Keyword" { Keywords } { .split "," .first .trim }
; returns a new table with additional column

spr .add-column "Fullname" { A B } { A + " " + B }
```

You can also add colunn using averf

```lisp
spr .add-column regexp ""
```
&nbsp;

### Droping a column

Adding columns vs. changing existing ones is I think a clearer strategy, but you could end up with a bunch of columns you don't want


```lisp
spr .add-column "Main Keyword" { Keywords } { .split "," .first .trim }
; returns a new table with additional column

spr .add-column "Fullname" { A B } { A + " " + B }
```
&nbsp;

### Grouping and Aggregates

```lisp
spr .group-by 'Species_cln { }
; 
```
&nbsp;

### Joining tables together

```lisp
; load another

; left join

; inner joun
```
&nbsp;


## Table as state store

### Ref Table

```lisp
rspr: ref spr
```

...

```lisp
rsps .rename-column 'Adversary "Adversary name"

```lisp
rsps .rename-column 'Adversary "Adversary name"

...
```


## Saving a table

### As CSV

```lisp
spr .save\csv %file.csv

spr .save\xlsc %file.xlsx

spr .to-json\lines %file.lines.json

spr .dump .write* %file.rye


...
```

