---
title: "Values and assignment"
date: 2019-02-11T19:30:08+10:00
draft: false
group: true
weight: 200
summary: "About values and assigning them to words."
mygroup: true
---

_"values, values everywhere ..."_

## Values

All that Rye is, is a bunch of Rye values. All your code is made of Rye values and all your code works with are Rye values.

Some values are atomic. For example integers, decimals, text, emails, URLs, words ...


```clojure
33                     ; integer
3.14                   ; decimal
"Hello word"           ; text
jim@example.com        ; email
https://example.com    ; URL
'word                  ; literal word
```

Also a Rye value, a Block, is a colections of values.

```clojure
{ 1 2 3 }              ; block of integers
{ "Hello" "Word" }     ; block of texts
{ 33 "Jane" admin }    ; mixed block with integer, text and a word
{ apples { 4.4 3.2 } } ; block including another block
```

All values above are **literal values**, meaning you can enter them directly in code. But Rye also suports many types of constructed vales. But firs, let's look at how we assign values to words.

## Assignment

We use set-words to assign values to words. You recognise a **set-word** by a collon ":" on the right.

```clojure
age: 33
pi: 3.14
message: "Hello World"
email: jim@example.com
website: https://example.com
type: 'word
users: { { 33 "Jane" admin } { 42 "Jim" user } }

; we cal also assign results of expressions
; we haven't yet learned about calling functions
; but you will probably understand these two
age: 33 + 1
block: first { 1 2 3 }
```

## Constructed values

Constructed values are _constructed_ by **functions**. Most common constructed values
are contexts and functions. Functions are like other rye values and are created by a **fn** (or similar) function.

```clojure
person: context { name: "Jim" age: 45 }  ; creates a context
printer: isolate { prn: ?print }         ; creates an isolated context

say-hello: does { print "Hello!" }       ; creates a function with no arguments
greet: fn { n } { print "Hi, " + n }     ; creates a function
square: pfn { x } { x * x }              ; creates a pure function 

; lists and dicts are collections of unboxed values that only get boxed lazily.
; Imagine you are loading a list of 1 million JSON objects. If we don't prematurely
; turn all values to Rye values (box them) we can load them as fast as Go does.
; Then we can select a smaller section of them, or aggregate them in this raw mode. 
; So Rye directly usually only needs to work with a subset of that full data.
user: dict { "id" 103 "name" "jime" }    ; creates a dict
primes: list { 2 3 5 7 11 13 }           ; creates a list
```

<!-- Conctructor functions are no different than any other function. Their only difference is in functionality and we try to name them consistently.-->

----
**Bonus material**

### Inlining set-words

Everything in Rye is also an expression, returns a value. Setwords also return the assigned value. Not saying it's a good idea to abuse, but they can be used inline.

```clojure
fruits: 100 + apples: 12 + 21
; we set:
; apples: 33
; fruits: 133
```


### Left set-words

You will se _why_ later, but Rye also has left leaning set-words.

```clojure
"Jim" :name
12 + 21 :apples + 100 :fruits
```
