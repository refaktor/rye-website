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
"Hello word"           ; text
https://example.com    ; URL
'word                  ; literal word
{ 1 2 3 }              ; block of integers
{ "Hello" "Word" }     ; block of texts
```

## Set-words

And these values can be assigned to words using set-word, words that have a colon or their right.


```clojure
age: 33
message: "Hello World"
website: https://example.com
type: 'word
numbers: { 1 2 3 }
words: { "Hello" "World" }
```

If you've programmed in any language before, the concept is the same as with variables.

## Using bound words

We haven't looked at function calls yet, but the use of `print` function below will be self-evident I hope.

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

Again we haven't looked at op-words yet, but effect of + is probably understandable.

```clojure
meaning: 24 + 18

print meaning
; prints: 42
```

## Inline use

Everything in Rye returns a value. Use of Set-words also return the assigned value, so they could be used _inline_ of expreee.

```clojure
fruits: 100 + apples: 12 + 21
; we set:
; apples: 33
; fruits: 133
```

## Left set-words

You will se _why_ later, but Rye also has set-words that take value from the left.

```clojure
"Jim" :name
12 + 21 :apples + 100 :fruits
```
