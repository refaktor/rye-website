---
title: 'Loops' 
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 500
summary: "Repeating, looping, iterating in Rye."
mygroup: true
---

## Just functions again

Like conditionals (if, either, switch) looping _constructs_ in Rye are also just functions. Rye has many of them, and you can create your own.

## Simple loop

Sometimes you just need to loop N times. And there is a function for that. It accepts integer - a number of loops and a block of code.

```clojure
loop 3 { print "Hey!" }
; prints:
; Hey!
; Hey!
; Hey!

loop 2 { prns "Bye" }
; prints: Bye Bye
```

The use of left set-word might seem a little odd, but you will see the benefits later.

## For function

### Injected values - quick crash course

We are getting ahead of ourselves. We will learn about injected values, op-words and similar things later, but we need to touch them a little. We could skip them at loop
function, but the next one (for) doesn't make any sense without them.

```clojure
123 :a                    ; you already learned about left leaning set-word
print a                   ; it gets its value from the left
; prints: 123

loop 3 { :i , prns i }    ; loop function injects loop number into the 
                          ; code block and left set-word can pick it up
; prints: 1 2 3

with 100 { :x , print x } ; with takes an argument and injects it. Now why
                          ; would you do that? There is more to this as
						  ; you will see later
; prints: 100
```

There are other functions that inject values into blocks, not just loop, with (and for). There are also other elements of the language that let you use that.

BTW: comma is called expression guard and is optional.

### Now a "for" loop

For functions takes a block of values and a block of code. It iterates through values passing each as injected value into the block as it evaluates the block.

```clojure
names: { "Jim" "Jane" "Anne" }

for names { :name , print "Hi " + name }
; prints:
; Hi Jim
; Hi Jane
; Ji Anne

; range takes two integers and creates a block of integers between them 

for range 1 5 { :i , prns i }
; prints: 1 2 3 4 5 
```

<!--
### Op-words

We will come to them later, but we would usually write the examples above with op-words. When function is entered as op-word it takes the first argument _from the left_. 
So the injected value is directly taken as fist (and in these cases only) argument of function inside block. 

```clojure
loop 3 { .prns }
// prints: 1 2 3

for names { .printv "Hi {}" }
// prints:
// Hi Jim
// Hi Jane
// Hi Anne

for range 1 5 { .prns }
; prints: 1 2 3 4 5 
```
-->
