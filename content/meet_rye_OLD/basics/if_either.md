---
title: 'Conditionals'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 400
summary: "About conditional execution, your if and else."
mygroup: true
---

_"if it has no special forms, how does it have if"_

## Revisit Do again

On the previous page we looked at the **do** function. It accepts one argument, a block of code and evaluates it.


```clojure
do { print "Hello" }
; prints: Hello
```

## If

Now think of a function similar to **do** function. But it takes another argument and only evaluates the block of code if the first argument is True (1). 

And so we get **if**, and it's just an ordinary function.

```clojure
if 0 { print "Moon" }
; doesn't print anything

if 1 { print "Sun!" }
; prints: Sun!

if 10 < 20 { print "Hello" }
; prints: Hello

value: 101

if is-integer value { print "Great!" }
; prints: Great!
```

## Either

To achieve if-else behaviour we have another function called **either**. 

It accepts conditional and two blocks. If condition is true it evaluates first block, if not the second.

```clojure
either 1 { print "hello" } { print "yello" }
; prints: hello

either 0 { print "hello" } { print "yello" }
; prints: yello
```

Like everything else in Rye either is an expression and returns something. It returns the last value in the block it evaluates. So it acts as a ternary operator would in C-like languages.

```clojure
print either 1 { "hello" } { "yello" }
; prints: hello

num: 99
increment: true              ; true is just a function that returns 1
print num + either increment { inc num } { num }
; prints: 199
```

## Switch

**Switch** is also a function in Rye. Similar to **either** it returns the last value of evaluation.

```clojure
switch 2 { 1 { print "one" } 2 { print "two" } }
; prints: two

animal: "duck"

print switch animal { "duck" { "quack" } "dog" { "woof" } }
; prints: quack

```

---
-- Bonus --

### BONUS: Code blocks are Rye values

Blocks of code are just like other Rye blocks, we can store them in a variable for example. There is nothing special about **if**, it's just a function. 

```clojure
say-hello: { print "hello" }

if 10 > 20 say-hello
```

<!-- ### BONUS: Everything is an expression

All these are expressions that return the result of the evaluated block so Rye's way is more like:

```clojure
print either 10 > 20 { "hello" } { "yello" }
; prints: yello

print switch 2 { 1 { "one" } 2 { "two" } }
; prints: two
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
-->
