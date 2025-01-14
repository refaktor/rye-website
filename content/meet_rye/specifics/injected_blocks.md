---
title: 'Injected blocks'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 500
summary: "Injecting values in blocks that evaluate."
mygroup: true
---

We've been dancing around this language feature so far, but it's such a natural way to code in Rye that it's used a lot and we couldn't escape it when showing how to use `for` loop, so we already did a little [crash course](). 

But we also didn't know op-words then, so we could use just left leaning set-words. Now we will finally be able to use full Rye.

## With function

All code in Rye lives inside blocks, and Rye functions and assignment can take values from the left, so it makes sense that when we start evaluating a block, that we can sometimes already inject a value that Rye code can pick up and use.

```clojure
msg: "Hey"

; do evaluates a block of code. It doesn't inject any value.

do { print msg }

; with takes another argument and it injects that value into block when it evaluates it

with msg { .print }
```

## Using injected blocks

### Loop function

Let's return back to the **loop** function with our knowledge of injected blocks and op/pipe words. Now take this example from before:

```clojure
loop 3 { ::i , prns i }
; prints: 1 2 3
```

becomes:

```clojure
loop 3 { .prns }
; prints: 1 2 3
```

### For function

Similar for the **for** function. We had this before:

```clojure
names: { "Jim" "Jane" "Anne" }

for names { ::name , print "Hi " + name }
; prints:
; Hi Jim
; Hi Jane
; Ji Anne

for range 1 5 { ::i , prns i }
; prints: 1 2 3 4 5 
```

and now we have:

```clojure
names: { "Jim" "Jane" "Anne" }

for names { .printv "Hi {}" }
; prints:
; Hi Jim
; Hi Jane
; Ji Anne

for range 1 5 { .prns }
; prints: 1 2 3 4 5 
```

### Flow combinators

More functions use the injected block mechanism. Some, like `pass`, `keep` and `wrap` are called flow combinators. They aren't used that much, but they come in handy here and there.

```clojure
; pass evaluates the block, and then returns/passes the first argument forward

101 .pass { * 3 |print } |* 5 |print
; prints:
; 303
; 505

; keep evaluates the first block, then the second, but passesforward the result of the first

4 .keep { * 11 } { + 11 |print }
; prints: 15
; returns: 44
```

### Higher-order functions (*like)

Rye supports a great number of functions for patterns that are usually called higher-order functions (filter, map, reduce). 

In Rye, these functions can just take blocks of code (like `if`, `loop` or `for` functions) for what you would need special forms in most languages. 

Rye really has a lot of these, see the [reference](structures.html#heading-Higher%20order%20like%20functions) for them. And here both injected blocks and op/pipe words come very handy.


```clojure
nums: { 1 2 3 4 5 }

map nums { + 10 }
; returns: { 11 12 13 14 15 }

filter nums { > 2 }
; returns: { 3 4 5 }

reduce nums 'acc { + acc }
; returns: 15

seek nums 'acc { .is-even }
; returns 2

partition nums { > 2 }
; returns { { 1 2 } { 3 4 5 } }

; example from before, so you won't think HOFs are 
; useful just for swaping numbers

text: "Did snake eat the apple?"
words: { "apple" "snake" }

fold words 'tx text { .replace* tx "****" }
; returns: "Did **** eat the ****?"
```

### Functions

As explained before, we have more than one _function_ for constructing functions. `fn1` is a function that takes one anonymous argument, and injects it into the function block. Either way, the first argument is always injected into the function block.

```clojure
add10: fn { a } { a + 10 }

; is the same as
add10:: fn1 { + 10 }

; this also works
add: fn { a b } { + b }
```

## Expression guards

The use of a comma inside a block is an optional **expression guard**. You don't need to use it, but sometimes it's better to be certain and visually cue the border of expressions.

```clojure
x: 42

do { print x  x + 1 }
prints: 42
returns: 43

; expression guard is entirely optional but sometimes beneficial
do { print x , x + 1 }
prints: 42
returns: 43
```

But the comma also has another effect in injected block, since we explicitly determined the border of previous expression, we **re-inject the injected value** after it. This little behaviour gives us a lot of additional flexibility in combination with functions that work with injected blocks.

```lisp
33 .with {
  * 10 |print ,
  + 10 |print
}
; prints:
; 330
; 43

{ 7 4 12 21 } .with {
  .max .printv "Max: {}" ,
  .min .printv "Min: {}" ,
  .avg .printv "Avg: {}" ,
  print "---------" ,
  .sum .printv "Sum: {}"
}
; prints:
; Max: 21
; Min: 4
	; Avg: 11.0
; ---------
; Sum: 44
```
