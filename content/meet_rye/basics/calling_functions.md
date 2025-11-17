---
title: "Calling functions"
date: 2019-02-11T19:30:08+10:00
draft: false
group: true
weight: 200
summary: "Because every active part in rye is a function, all we do in Rye is call functions."
mygroup: true
---

_"all day I sit here and make function calls"_

## Print hello

When you are writing programs in Rye, almost all that you actually _do_ is make function calls. Well, you do that in every language, but in Rye, this statement is much more true than in most of them. More on that in the next page.

Many functions are already built into Rye runtime, they are written in its host language Go. We call them **built-in functions** or **built-ins** for short. Other functions, just **functions**, are written, or you write in Rye. You
call both the same way.

One such built-in function is **print**. It prints a value to standard output and goes to the next line.


```clojure
print "Hello!"
; prints: Hello!

print "Hello World!"
; prints: Hello World!
```

The **print** function takes one value (an argument) and prints it. We call it by naming it and providing an argument, in this case some text to print.

## Return values

Print or some variant of it (println, echo, ...) exists in almost every language, but most functions in Rye are not like print. We don't call them for effect, but for their return value.

**Inc** (increment) accepts one integer value and returns a value that is an increment of the accepted value. It doesn't change anything and for the same input (the argument), it always returns the same result. 

```clojure
; Functions taking one argument
inc 99
;returns 100

first { 1 2 3 }
; returns: 1

avg { 8 9 11 12 }
; returns: 10.0

join { "Hello" "World" }
; returns: "HelloWorld"
```

As we said, a big part of built-in functions are like that, and many of them take more than one argument.

```clojure
; Functions taking two arguments
_+ 123 123
; returns 246

factor-of 10 2
; returns 1

tail { 1 2 3 4 5 } 2
; returns: { 4 5 }

intersection { 1 2 3 4 5 } { 0 2 4 6 }
; returns: { 2 4 }

join\with { "Hello" "World" } " "
; returns: "Hello World"
```

All the lines above are expressions, because they return results and have no side effects. We then combine expressions to get the desired result.
<!-- We create programs by combining functions like above and adding some functions that affect the state, print things, load and save them to disc, display. -->

## Words with values

In the previous page, we learned that we can assign values to words. Invoking a word is the simplest expression. If the value is not a function, it returns the assigned value.

```clojure
name: "Jane"
; we assigned text "Jane" to word name

name
; returns: "Jane"

print name
; prints: Jane
```

## Function composition

Since function calls produce a result, we can compose them together. A result of one function call can be an argument to another.

```clojure
print inc 999
; prints: 1000
```

Let me tell you a story:

* Rye finds a word **print**. It sees it's bound to a function that takes one argument, so it goes forward looking for one
  * there it finds a word **capitalize**, also bound to a function with one argument. So it again goes forward seeking
    * it finds a word **name** which is bound to a text value "bob", it takes it and passes it to **capitalize**
  * capitalize returns text "Bob" which it now passes back to **print**
* and the function behind **print** displays it on the screen

```clojure
name: "bob"

print capitalize name
; prints: Bob
```

<!-- Constructor functions are no different than any other function. Their only difference is in functionality and we try to name them consistently.-->

## Convention: Function variants

In Rye (like in REBOL) all functions have a fixed number of arguments. REBOL had something called refinements to remedy this, the idea was good on the first level, but you couldn't elegantly pass refinements to next calls. Refinements also 
clashed syntactically with context navigation. So Rye doesn't have this concept. 

It has a naming convention. Where there is a primary function, but you need more variations or specializations of it, we use a "\\" character in the name of the function to denote that. You saw one example above with join and join\with. Remember these are
two different functions, it's just a naming convention.

```clojure
; join & variants

join { "I" "am" 100 "%" "sure" }
; returns: "Iam100%sure"

join\with { "I" "am" 100 "%" "sure" } " "
; returns: "I am 100 % sure"

; print & variants

print { "Jane" "Bob" "Anne" }
; prints: { Jane Bob Anne }

print\csv { "Jane" "Bob" "Anne" }
; prints: Jane,Bob,Anne

print\json { "Jane" "Bob" "Anne" }
; prints: [ "Jane", "Bob", "Anne" ]

; map & variants

map { 9 99 999 } { + 33 }
; returns: { 42 132 1032 }

map\idx { 10 100 1000 } 'i { * i } ; Rye distinguishes between index (offset)
; returns: { 0 100 2000 }

map\pos { 10 100 1000 } 'i { * i } ; and position (first, second, ... )
; returns: { 10 200 3000 }
