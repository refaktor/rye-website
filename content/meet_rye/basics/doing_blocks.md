---
title: 'Do-ing blocks'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 300
summary: "All Rye code resides in blocks. Block is the main building block."
mygroup: true
---

_"to do or not to do, is now the question"_

## REBOL and Lisp

In REBOL, contrary to Lisps, blocks or lists don't evaluate by default. For better or for worse, this little difference is what makes REBOL - REBOL.

Fire up a random online Lisp REPL and enter:

```lisp
; Lisp

(print (+ 11 22))
; prints: 33            -- evaluated the list and printed the result

(print '(+ 11 22))
; prints: (+ 11 22)     -- we had to quote it so it doesn't evaluate the list
```

When we give Lisp a block it evaluates it and prints the result. If we want to print the block, we need to quote it. Lisp **evaluates by default**.

## REBOL and Rye

Rye came out of REBOL's ideas and this is one of the core ones. So Rye and REBOL work the same here. What happens if we do the same in Rye shell ...

```clojure
; Rye

print { _+ 11 22 }
; prints: { _+ 11 22 }  -- prints the block, doesn't evaluate

print do { _+ 11 22 }
; prints: 33            -- we had to *do* it, to evaluate it
```

Just the contrary than in lisp. If we give the print function a block, it doesn't evaluate it but prints it, and if we want to evaluate it, we have to **do** it. Rye **doesn't evaluate by default**.

## Function Do

Rye has a function **do** that accepts one argument, a block of code. Do you remember we said that normal Rye is a **Do dialect**, this is why.

```clojure
do { print "hello" }
; prints: hello
```

Function **do** returns value of the last expression in the block.


```clojure
do { print "hello world" }
; prints: hello world
; returns: "hello world"


do { print "hello world" 101 + 10 }
; prints: hello world
; returns: 111
```

## Blocks of code are just blocks

Blocks of code are just like other Rye blocks.

```clojure
who-are-u: { print "James" "Bond" }

print first who-are-u
; prints: print

print do print who-are-u
; prints 3 lines: 
; { print "James" "Bond" }
; James
; Bond
```

Everything in Rye returns something, so print returns the value it's printing.
