---
title: 'Returning words*'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 900
summary: "Rye doesn't have a return keyword, but it has function that can also return. What?"
mygroup: true
---

_"I'll return back" Ryeminator_

## Implicit return

Like Lisps, Rebol and most functional languages Rye has an **implicit return** behaviour. We generally don't use the return "keyword" to return from function, but the value of last expression in a function is **always** returned.

Remember, everything is Rye is an expression - i.e. returns something.

```clojure
add: fn { a b } { a + b }
print add 18 24
; prints: 42

greet: fn { direction } { either direction = 'in { "Hello" } { "..." } }
print greet 'in
; prints: Hello
print greet 'out
; prints: ...

greet2 fn { dir } { switch dir { in { "Hi" } out { "Bye" } _ { "Whaa" } } }
print greet2 'in
; prints: Hi
print greet2 'out
; prints: Bye
print greet2 'up
; prints: Whaa
```

## Return function

While **rarely used** Rye has a return **function**. But how can it be a function? If you read the basics, all active components of Rye are functions and so it return. Functions if Rye can cause a return to caller and return is one of such functions.

This is not the usual way, but we could write the same thing as above using *if* and *return* functions.

```clojure
greet: fn { direction } { if direction = 'in { return "Hello" } "..." }
print greet 'in
; prints: Hello
print greet 'out
; prints: ...

find-num: fn { list n } { .for { = n |if { return true } } false }
{ 1 3 7 13 } .find-num 7
; returns 1
{ 1 3 7 13 } .find-num 8
; returns 0
```

## Returning functions

```clojure
```



## Uses of returning words

```clojure
```
