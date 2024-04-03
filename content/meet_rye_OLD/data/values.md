---
title: Rye values
weight: 100
summary: "Lets look at atomic values Rye is composed of."
---

## Atomic values

Atomic values are values that in general terms can't be subdivided (1), or more exactly that represent one, singular value. The simple atomic values are:

```clojure
1                   ; integer number
3.14                ; decimal number
"Jane"              ; string
jane@example.com    ; email
https://ryelang.org ; uri
%foo/readme.md      ; file path
```

## Compound values

There are also multiple compound values. More specialized ones Lists, Dicts, Contexts, Spreadsheets, .... But the main and most important one is a Block. It's similar to a List or Array in other languages. 

It's Rye's main building _block_. All Rye values (and code) resides inside Blocks.

```clojure
{ 1 3.14 "Jane" jane@example.com  }  ; a block of four values

{ { "Jane" 320 } { "Jim" 290 } }     ; a block of two blocks

{ red green blue }                   ; a block of 3 words

{ print 101 + 10 }                   ; a block of what looks like code

```

These two groups of data types can solve a lot of your data definition needs. But another important group can help also.

---
-- bonus --

## Words

Word is another important data type. In a programming context words are bound to other values, but in data context, words are just words. They are indexed entities, you can imagine them as Enums.

Below are 4 valid words:

```clojure
a-word

is-this-a,word?

a-bag-of[apples]

(first-to-last)
```

Words can use a lot of characters (but not all). Here we might mention an important rule about Rye. **Each token in Rye code must be separated by spaces from the next one**.


## Special word types

Rye has many types of words. It might seem a little strange at first, but you will see that they can get useful.

```clojure
word      ; normal word
'word     ; lit-word
word:     ; set-word
:word     ; lset-word
?word     ; get-word
.word     ; op-word
|word     ; pipe-word
```
---

(1) strings can be split to multiple strings, uri-s and filepaths to separate parts, heck, even numbers can be divided :)
