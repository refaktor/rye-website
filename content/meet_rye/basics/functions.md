---
title: 'Functions' 
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 600
summary: "Creating your functions."
mygroup: true
---

## Builtin functions

Rye comes with many builtin functions. Functions that are defined in a host language (Go). While solving a problem you build functions of your own in Rye.

## Rye functions

Functions are like any other Rye values. You create them by calling a function. The most common is builitn function *fn*. It takes two arguments a block of arguments and a block of code.

It returns a function value which we assign to a word using a set-word, like any other value.

```clojure
say-hi: fn { name } { print join { "Hi " name } }

double: fn { x } { x * x }

is-positive: fn { n } { if n > 0 { return 1 } 0 }
```

Value of the last expression is retuned from function call. You don't need to use return statement (there is return function, for when you want to return in the middle of code).

We call them like we call builtin functions, by naming them and providing the arguments.

```clojure
say-hi "Jane"
; prints: "Hi Jane"

double 5
; returns: 25

is-positive -100
; returns 0
```

## Pure functions

If you **probe** builtin functions in shell you will see that most of them show something like `[Pure BFunction ...]`. A function being Pure means that it has no side effects and that
it has referential transparency. This means that with the same inputs it will always produce same output. 

Above **join** is a pure builtin functions, while print has a side effect, and function like `now` is not referentially transparent. It will return a different result each time you call it.

```lisp
probe ?join
; prints: [Pure BFunction(1): Joins Block or list of values together]

probe ?print
; prints: [BFunction(1): Prints a value and adds a newline.]

probe ?now
; prints: [BFunction(0): Returns current Time.]
```

Function for creating pure function is **fnp**. Pure functions can only call other pure functions (builtin or normal). Pure functions are bound to a context with just pure functions so they 
have no access to any unpure function or a word (variable) outside it's scome (like global).

```clojure
x: 101

add5: pfn { x } { x + 5 }
add5 10
; returns: 15

addx: pfn { x } { x + a }
addx 10
; Error: word a not found

say-hi: pfn { name } { print "Hi" + name }
say-hi "Anne"
; Error: word print not found
```

BTW: `+ * >` we used above are also pure functions. One or two letter operator characters are _op-words_ by default. More on this later.


## Functions with no arguments

Continuing REBOL's convention function that accept no arguments can be created with **does** function. 

```clojure
say-hi: does { print "Hi!" }

say-hi
; prinsts: Hi!
```

Since there is no special keyword like: `func, def, function, ...`, but functions are
like other Rye values constructed with functions (and you can load these as a library or make your own), we tend to use more words for specific cases of constructing functions than just one keyword.

## Closure

It turns out closures are just a simpler variant of this. Closure is a function that's executed in the context it was defined in.

```clojure
me: context { 
	name: "Jim" 
	intro: closure { } { print { "I'm" name } 
}

name: "Joe"

me/intro
; prints: I'm Jim

; also i we take closure out it's context and assign it locally
me -> 'intro :my-intro

my-intro
; prints: I'm Jim
```
A more typical use of closures is that they are returned from functions while enclosing on function's context. Each function runs in its own context, so it's not that different than the example above.

```clojure
make-adder: fn { a } { closure { b } { a + b } }

add5: make-adder 5 
add10: make-adder 10

a: 30

add5 5
; returns: 10

add10 5
; returns: 15
```
