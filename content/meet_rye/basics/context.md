---
title: 'Contexts'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 600
summary: "Context or scope is how evaluator finds values bound to words."
mygroup: true
---

Context is akin to what's called scope in most programming languages. Context is environment in which Rye code is evaluated in. Context gives Rye words, well, a context. 

But context is also just another Rye value type. You can create them, manipulate them, link and combine them together, write functions that work with them or in them.

## Context structure

Context is just a dictionary of words and values, an optional link to the parent context and an optional dosctring.

## First look

When you start Rye console you enter an empty context. Your context has a parent context where all the builtin functions are defined. You can list the empty context with `lc` (list context) and a parent context `lcp` (list parent context) functions.
 
_BTW: Try the code below in the console on the top right, it's always better to see it work for real :)_

```clojure
lc
;Context:

lcp
; ...
; ... will print all builtin functions and subcontexts
; ...
; where-lesser: [BFunction(3): Returns table of rows where specific colum is lesser than given value.]
; where-match: [BFunction(3): Returns table of rows where a specific colum matches a regex.]
; where-not-contains: [BFunction(3): Returns table of rows where specific colum contains a given string value.]
; with: [BFunction(2): Takes a value and a block of code. It does the code with the value injected.]
; wrap: [BFunction(2): Accepts a value and a block. It does the block, with value injected, and returns (passes on) the initial value.]
; wrap\failure: [BFunction(2): Wraps an Error with another Error. Accepts String as message, Integer as code, or block for multiple parameters and Error as arguments.]
; xor: [Pure BFunction(2): Bitwise XOR operation between two values.]
```

Now we'll define two variables and a function. In fact, we bind three words with values, first word to an integer, second to a text and third to a function. Then list the current context again. 

```clojure
name: "Gerald"
xp: 1023
speak: does { print "Hmm" }

lc
; Context:
; name: [String: Geralt]
; speak: [Function(0)]
; xp: [Integer: 1023]

```


## Word resolution

Words are just words. The context in which we evaluate the code, consisting of words and literal values, gives them meaning (values).

When the evaluator gets to a word, it looks for word's bound value in the current context. If it doesn't find one, it goes up to the parent context, if a context has a parent, and looks there, and so on ...

In the parent context of our opening context, `print` is bound to a builtin function that prints a text. Evaluator finds the word and since it's value is a function, it evaluates the function.

```clojure

lcp\ "print"        ; list parent context with a filter
; Context:
;  print: [BFunction(1): Prints a value and adds a newline.]
;  ...

print "Hi"
; prints: Hi

print name          ; name is found in current context
; prints: Geralt

speak               ; speak is found in current context and is bound to a function
; prints: Hmm
```

## Setting context values

Evaluator can find _(read)_ a value from current or any parent context, but there is no syntax in Rye to directly set _(write)_ a value in parent or sub-contexts. 

Set and mod words always set or change values of words in the context code is evaluated in.

```clojure
name: "Jim"

friend: context { name: "Jane" , print name }
; prints: Jane

print name
; prints: Jim

print friend/name
; prints: Jane
```

## Sending messages

What you can do, is "send messages" to other contexts, that means "call functions" there. But in Rye a limited number of functions that change values in-place (hence can change state) need to end with "!" (exclamation mark), so the calls to such modifying functions are visible and explicit.

```clojure
count: 0

actuator: context { loop 100 { inc! 'count } }

print count
; prints: 100

reseter: context { change! 0 'count |if { print "Changed!" } }

print count
; prints: 0
```


## Function evaluation

Functions are evaluated in their own contexts and exactly the same rules apply. Functions can't change values outside their context directly, but they can read values from their parent contexts. If not, they also wouldn't have access to any other functions, because they  are also just words defined in (parent) contexts.

```clojure
name: "Jim"

change-name: fn { n } { name: n } ; does nothing outside the function

change-name "Bob"

print name
; prints: Jim
```

You can create functions in multiple ways and by this define which context is set as parent context or a direct context when the function evaluates. This is a source of many interesting features, we will look into this in the **Rye specifics** section.

## No global context

Lets review what we learned in the context of so called "global scope". 

In Python, for example, a function can modify global variables. You explicitly declare which variables inside a function are global with a `global` keyword.
In Rebol you explicitly define which 
words are local. This is problematic. If you forget to define a word as local, you are setting and changing a global variable. 

Both approaches mean that you have to _scan the code higher up_ in a function to understand, where _local code_ makes modifications.

**Rye functions can't change the global scope (context), or any other context, by assignment at all.**

```clojure
messages: { }

add-message: fn { m } { .append! 'messages }     ; will work

init-messages: fn { } { messages: { "First!" } } ; won't work, will set local messages

init-messages

add-message "Hello"

probe messages
; [Block: [String: Hello] ]
```

In fact, there is **no global scope** in Rye. There are no absolute scopes, all scopes (contexts) are **relative**. Individual contexts of separate linked chains of contexts and you can define a function to evaluate in any of them.


---

If you wish to read more about **contexts** right now, visit the **[first class contexts](../../specifics/context/)** page.



<!--
### Evaluation 

Let's do few very basic things and explain exactly how the words resolve. The code behaves as one would expect, so what I'll be explaining will be maybe painfully obvious, but it's good 
to be certain about the mechanism, not just assume it behaves as you imagine.

```clojure
print "Hi"
; prints: Hi
```

Evaluator arrives at word `print`, since it's not defined in current context it looks up the parent where it finds it bound to a builtin function accepting one argument.
It finds text "Hi" as the next value which it uses as the argument to builtin function `print` and the builtin function prints it. We can check that print really is defined
in a parent context.

```clojure
lsp\ "print"
; Context:
;  print: [BFunction(1): Prints a value and adds a newline.]
;  ...
```

Now we'll print a word defined in current context. Evaluator as before goes seeking an argument for the builtin function, but arrives at a word name. It looks in local context
first and finds it bound to text "Geralt" so it returns the text to the function at it prints it.

```clojure
print name
; prints: Geralt
```

Now we'll call a function `speak`. It's evaluated in it's own context but of which parent context is current context at runtime. We have plenty of options for it to be evaluated in context of it's creation, or any custom context or linked to a custom context.

So evaluator arrives as word speak. It finds it in current context bound to a function with no arguments. It evaluates it's body in it's own context. There is no print defined in it, so it looks at the parent context. There is no print there, but it is in the parent of the parent.

```clojure
speak
; prints: Hmm
```
-->

