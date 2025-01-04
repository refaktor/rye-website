---
title: 'Rigid about state*'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 300
summary: "In place changes are rare and visible, direct out of context changes are impossible."
mygroup: true
---

## Story time

I used to think that flexible language is just that. Flexible, which I liked for certain jobs. But it is also unpredictable, maybe eventually too 
dynamic for its own good.
And my experience confirmed that. REBOL was VERY flexible, but I could also change things I didn't want in places I didn't predict or think through 
all the way.

So when I returned after many years back to Python (and Java for mobile), I felt a sense of relief, that I'm back in a more boring, but safer world.

_I returned to Rebol again a after couple years again, Python at the time forced me to think too imperatively. I then coded REBOL with more reserve 
(I avoided runtime code manipulation) created the validation and SQL dialects, some tools to prevent my local variables from "escaping", etc ..._

## Separate things

Over the years I started seeing the difference. There is the language and there is the environment that language lives in and manipulates. 

* language flexibility: ability to compose words into richer _sentences_ **- good**
* environment flexibility: ability to more easily manipulate environment / values / state **- bad**

With Rye I try to increase language flexibility. I believe programming languages are nowhere as flexible as the languages we are all
already used to, programmers or not. Imagine we would communicate just with nouns and verbs, no teneses, counts, ...

But I also try to limit how you can modify the environment. 

## Environment limits

Rye imposes the folowing to make the changes you make more limited, explicit and visible:

* Rye values in general are **immutable** and Rye functions in general always **return new values** instead of modifying existing values
* Rye has some functions to change values in place but they all end with a "!"
* Rye can directly **only** change values in the **current context**. There is no syntaxt to change it in parent or sub context
* Rye set-word can only initialize values, if you want to modify them you have to use visually more costly mod-word so updates
  are visible and fewer in numbers

Let's see some examples

### Immutable values

Rye values on their own are immutable. I remember of using `list.append(1)` in Python a lot. `append` was a common function in Rebol too. I was
never really sure if I am changing the original list/bloc or the returnd value will have the item appended. In Rye we use `+` or `concat`
to return a new block with a new element.

```Python
family = [ "Anne", "Bob" ]

family.append("Joey")

for p in family { print p }
```
&nbsp;

In Rye we would do:

```lisp
family: { "Anne" "Bob" }

family:: family .concat "Joey"   ; or> family:: family + { "Joey" }

for family { .print }
```
&nbsp;

### Modifying functions

If you need them, for performance reason, Rye has option to change a block in place. You have to ref a block and the function that does it
has a "!" on end. You can use this if you have a concrete reason, but you wouldn't just be using this as default way to code. And in most
cases you don't need it.

```lisp
family: ref { "Anne" "Bob" }

"Joey" .append! family

for family { .print }
```

> **Q** Why is the order of arguments reversed? The active value is usually a first argument in Rye, so it can be passed down the pipe from
> previous operation
