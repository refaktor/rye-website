---
title: "Assignment"
date: 2019-02-11T19:30:08+10:00
draft: false
group: true
weight: 50
summary: "About assigning values to words."
mygroup: true
---

## Values

We've seen that Rye code can consist of literal values, for example:

```clojure
33                     ; integer
"Hello world"          ; text
https://example.com    ; URL
'word                  ; literal word
{ 1 2 3 }              ; block of integers
{ "Hello" "Word" }     ; block of texts
```

## Set-words

These values can be assigned to words using set-words, i.e. words that have a colon on their right.

```clojure
age: 33
message: "Hello World"
website: https://example.com
type: 'word
numbers: { 1 2 3 }
words: { "Hello" "World" }
```

If you've programmed in any language before, these would be the constants. The default assignment in Rye is to a constant.

## Using bound words

We haven't looked at function calls yet, but the use of the `print` function below will be self-evident I hope.

```clojure
print "Hello world!"
; prints: Hello world!

message: "Hello Mars!"
; assigns text "Hello Mars!" to word message

print message
; prints: Hello Mars!
```

## Assigning expression results

Values that get assigned to words can also come from expressions.

Again we haven't looked at op-words yet, but the effect of + is probably understandable.

```clojure
meaning: 24 + 18

print meaning
; prints: 42
```

## Inline use

Everything in Rye returns a value. Use of set-words also return the assigned value, so they could be used _inline_ within an expression.

```clojure
fruits: 100 + apples: 12 + 21
; we set:
; apples: 33
; fruits: 133
```

## Left set-words

You will see _why_ later, but Rye also has set-words that take a value from the left.

```clojure
"Jim" :name
12 + 21 :apples + 100 :fruits
```

## Variables and mod-words

All of the above assigns (binds) a value to a word and makes that word a constant in the current context, meaning, we can't later change it. This makes
code more predictable.

A word can also become a variable in its context by using a function **var** and a lit-word (literal word). Such words can be modified using 
the modifying functions (ending with a !) or a mod-word.

```clojure
; 5 minutes to midnight

var 'age 42

sleep ( 5 * 60 ) .seconds

; age: 43                              ; Would produce an Error

age:: 43

sleep 365 .days                        ; you would wait a while :)

change! 44 'age                        ; returns true if changed
```

Mod words are a little harder to look at, but that is by design. The goal is that you modify values under words only when you really need to.

The same holds true for the left mod-words. `::age`.
