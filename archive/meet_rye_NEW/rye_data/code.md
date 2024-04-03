---
title: 'Literal values'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 100
summary: ""
mygroup: true
---

## Simple values

In Rye you can define many types of values. The simplest ones are:

```clojure
1                   ; integer number
3.14                ; decimal number
"Jane"              ; string
jane@example.com    ; email
https://ryelang.org ; url
'domain             ; literal word
```

## Blocks

You can put Rye values in a Block - a block of values. Block is just another type of Rye value.

```clojure
{ 1 3.14 "Jane" jane@example.com  }  ; a block of four values

{ { "Jane" 320 } { "Jim" 290 } }     ; a block of two blocks
```

---


### Bonus: special word types

Rye has many types of words. It might seem a little strange at first, but you will see how useful they can be later.

```clojure
{
  word      ; normal word
  'word     ; lit-word
  word:     ; set-word
  :word     ; lset-word
  ?word     ; get-word
  .word     ; op-word
  |word     ; pipe-word
}```
