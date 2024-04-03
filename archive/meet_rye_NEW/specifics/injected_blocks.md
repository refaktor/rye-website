---
title: 'Injected blocks'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 500
summary: ""
mygroup: true
---

## No keywords, no special forms

There are no keywords in Rye. There are no special forms in Rye. It's all just application of functions and blocks.

## Blocks don't evaluate

Blocks don't evaluate on their own. This enables almost all that follows below.

```clojure
{ print "Hello" }
; returns a block of two values, a word print and a string hello
do { print "Hello" }
; prints: Hello
block: { print "Hello" }
do block
; prints: Hello
```

## If, either

If is a function with two arguments. A conditional and a block of code.

```clojure
if 10 < 20 { print "hello" }
// prints: hello
```
To achieve if-else behaviour we have function either, that accepts two blocks.

```clojure
either 10 > 20 { print "hello" } { print "yello" }
// prints: yello
```

## Switch

Switch is also a function in Rye

```clojure
switch 2 { 1 { print "one" } 2 { print "two" } }
// prints: two
```

---


### BONUS: Code blocks are Rye values

Blocks of code are just like other Rye blocks.

```clojure
say-hello: { print "hello" }
if 10 > 20 say-hello
```

### BONUS: Everything is an expression

All these are expressions that return the result of the evaluated block so Rye's way is more like:

```clojure
print either 10 > 20 { "hello" } { "yello" }
// prints: yello

print switch 2 { 1 { "one" } 2 { "two" } }
// prints: two
```

### BONUS: All these are just functions

If, either, switch are just library level functions, so we can have many of them and add our own ...

```clojure
// this would be a simpler way to achieve the specific switch solution
print select 2 { 1 "one" 2 "two" }
// prints: two

// from Rebol legacy we also have the case function
x: 2 y: 1
case { x = 1 { "boo" } all { x = 2 x = 1 } { "hoo" } } |print
// prints: hoo

// more as an experiment I also created cases function
for range 1 100 { :n
  cases ""
    { n .divides 3 } { "Fizz" }
    { n .divides 5 } { + "Buzz" }
    _ { n }
  } |prn
}
// outputs: 1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz 16 ...

// oh, and I see you just meet some *pipe-words*
```
