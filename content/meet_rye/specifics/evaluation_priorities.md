---
title: 'Evaluation priorities'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 310
summary: "How Rye evaluates expressions with different word types."
mygroup: true
---

Rye's base dialect has no special forms so all code evaluates in exactly the same manner based just on types of words in a block. 

Different types of words, and nothing else, do affect evaluation precedence, so understanding these basic rules will help you understand the code.

## The three ways

There are three main ways to call functions, each with different evaluation behavior:

```clojure
; regular words - collect arguments from the right
_+ 1 _* 2 3
; evaluates as: 1 + (2 * 3) = 7

; op-words - take first argument from the left
1 + 2 * 3
; evaluates as: 1 + (2 * 3) = 7

; pipe-words - wait for left side to fully evaluate
1 |+ 2 |* 3
; evaluates as: ((1 + 2) * 3) = 9
```

## Op-words: right to left

The behavior described above means, Op-words (like `+` or `.inc`) evaluate from right to left. Each op-word grabs the immediate value on its left.

```clojure
1 + 2 * 3
; first: 2 * 3 = 6
; then: 1 + 6 = 7
```

## Pipe-words: left to right

Pipe-words (like `|+` or `|inc`) wait for all expressions on the left to evaluate first, then take the result.

```clojure
1 |+ 2 |* 3
; first: 1 + 2 = 3
; then: 3 * 3 = 9
```

## Mixing them

You can mix op-words and pipe-words to control exactly how expressions evaluate:

```clojure
1 + 2 |* 3
; op-word + grabs 2, pipe-word |* waits for 1 + 2
; result: (1 + 2) * 3 = 9
```

## Using parentheses

When in doubt, parentheses give you explicit control:

```clojure
( 1 + 2 ) * 3
; result: 9
```

## With assignment

Set-words and mod-words have even **higher priority** than pipe-words, meaning they capture everything that follows until the end of expression (including pipe-words).

### Set/Mod-words (right assignment)

```clojure
x:: 1 + 2 * 3
; x = 7 (captures the whole expression)

x:: 1 |+ 2 |* 3
; x = 9 (captures the whole expression)
```

### Left Set/Mod-words (left assignment)

Left set-words (mod-words) capture the value from left in the same manner as pipe-words do:

```clojure
1 + 2 * 3 ::x
; x = 7 (captures the result)

101 ::x + 1 |+ 2 |* 3 ::y
; x = 101, y = 312
; ::x captures 101, ::y captures the final result
```

### Combining both

You can use both set/mod-words and left set/mod-words together:

```clojure
x:: 101 + y:: 1 |+ 2 |* 3
; x = 110, y = 9
; y:: captures the pipe expression (9)
; x:: captures 101 + 9 = 110
```

In aggregate: **op-words evaluate right-to-left**, **pipe-words evaluate left-to-right**, and **set/mod-words have higher priority**, capturing their respective expressions.
