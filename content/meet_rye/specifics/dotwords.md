---
title: 'Dot-words'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 295
summary: "Words that take their first argument from the left."
mygroup: true
---

## What are dot-words

Dot-words are words prefixed with a dot. They call the same function as the regular 
word, but take the first argument from the left instead of the right.

```clojure
inc 100        ; regular word - argument from the right
; returns: 101

100 .inc       ; dot-word - argument from the left
; returns: 101
```

## Chaining transformations

Dot-words enable left-to-right chains of transformations:

```clojure
"hello world" .upper .replace "WORLD" "RYE" .length
; returns: 9
```

## With multiple arguments

Only the first argument comes from the left. Additional arguments follow normally:

```clojure
"Hello World" .replace "World" "Mars"
; returns: "Hello Mars"

{ 1 2 3 4 5 } .tail 2
; returns: { 4 5 }
```

## Second argument from the left

Add a star `*` to take the second argument from the left instead:

```clojure
"apple" .replace* "the apple fell" "orange"
; returns: "the orange fell"

"world" .concat* "hello "
; returns: "hello world"
```

## Relationship to op-words

Dot-words are the general form of op-words. Operator characters like `+ - * /` are 
op-words by default (no dot needed). See [Op and Pipe-words](/meet_rye/specifics/opwords) 
for details on evaluation order.
