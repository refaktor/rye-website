---
title: Values
weight: 100
summary: "Take a look at basic value types in Rye."
---

_"Rye code is a block of Rye values. Block is also a Rye value."_

## Simple values

Rye code can represent various value types. Here are a few:

```
1                   ; integer number
3.14                ; decimal number
"Jane"              ; string
jane@example.com    ; email
https://ryelang.org ; uri
%foo/readme.md      ; file path
blue                ; word
context/word        ; cpath (context path)
```

## Blocks

Multiple compound value types exist in Rye, but a block type is the most important one. 

It's similar to a List in other languages. It's the main building _block_ for Rye code.

```lisp
{ 1 3.14 "Jane" jane@example.com  }  ; a block of four values

{ { "Jane" 320 } { "Jim" 290 } }     ; a block of two blocks

{ red green blue }                   ; a block of 3 words

{ print 101 + 10 }                   ; a block of what looks like code

{ if x = 0 { print "ZERO" } }        ; another block that looks like code with a sub-block
```

## Words

Word is another very important value type. In a programming context words are bound to other values, but in data context, words are just words. They are indexed entities.

All Rye entities **must be separated by spaces**, and many characters can be used for words. Below are 3 valid words:

```clojure
a-word

is-this-a...word?

(first-to-last)
```

## Word types

Rye has many types of words. It might seem a little strange at first, but you will see that they can be useful.

```clojure
word   ; word (evaluates to a bound value, if it's a func. calls it)
'word  ; lit-word (literal word, evaluates to a word)
word:  ; set-word (binds a value on the right to a constant word)
:word  ; lset-word (binds a value on the left to a constant word)
word:: ; mod-word (binds a value on the right, to a variable word)
::word ; lmod-word ( ... left ... )
?word  ; get-word (always returns the bound value, also when func.)
.word  ; op-word (evaluates a func. taking first arg from left)
|word  ; pipe-word (same, but evals all expressions on the left first)
```
