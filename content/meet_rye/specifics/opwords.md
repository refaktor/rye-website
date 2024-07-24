---
title: 'Op and Pipe-words'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 300
summary: "A different ways to call regular functions that enable left to right code flow."
mygroup: true
---

And we are finally here. First noticeable thing that probably hit you in the face if you looked at various Rye examples.

## Calling functions

Calling functions in Rye is simple. You use a word, that function is bound to, and provide the arguments.

When evaluator gets to a word bound to a function, it goes forward collecting necessary number of values for arguments and then evaluates the function. Here are few simple examples:

```clojure
print "Call me"
; prints: Call me

inc 100
; returns: 101

concat "Fizz" "Buzz"
; returns: "FizzBuzz"

factor-of 15 3
; returns: 1

replace "Hello World" "World" "Mars"
; returns: "Hello Mars"
```

## Calling functions with op-words

Now we will call the same functions with the same arguments, but we'll use op-words instead of regular words. Op-words begin with a dot. In case of an op-word, first argument 
is taken from the left.

```clojure
"Call me" .print
; prints: Call me

100 .inc
; returns: 101
```

As we see in next examples, only the first argument is taken from the left. The rest follow like with regular words. 

```clojure
"Fizz" .concat "Buzz"
; returns: "FizzBuzz"

15 .factor-of 3
; returns: 1

"Hello World" .replace "World" "Mars"
; returns: "Hello Mars"
```

## Operators

What are operators then. Operators in Rye are nothing special, they are just regular functions usually taking two arguments. The only difference is that one and two letter operator characters are by default recognized as **op-words**, they don't need 
the dot in front. 

The regular spelling of the word in this case is with added underscore in front op-word. For op-word `+`, regular word is `_+`.

```clojure
_+ 3 2                  ; _+ regular word
; returns: 5
_* 3 2
; returns: 6

3 + 2                   ; + op-word
; returns: 5
3 * 2
; returns: 6
```

## Pipe-words

Pipe words behave similarly or in simple cases exactly the same. So you will get the same results in these examples if as we got them with op-words:

```clojure
"Call me" |print
; prints: Call me

100 |inc
; returns: 101

"Fizz" |concat "Buzz"
; returns: "FizzBuzz"

15 |factor-of 3
; returns: 1

"Hello World" |replace "World" "Mars"
; returns: "Hello Mars"

3 |+ 2
; returns: 5
3 |* 2
; returns: 6
```


## The difference

The difference is in how they combine into larger _sentences_, how they combine with values, regular words and other op/pipe-words. 

Op-word applies to the first value it can on the left, and pipe-word lets all expressions on the left evaluate and then takes the result as the first argument.

This simple example shows the difference:

```clojure
13 + 14 .print |print
; prints:
; 14
; 27
```

More examples

```clojure
inc 10 * 10
; returns: 101

inc 10 |* 10
; returns: 110

"hi " .concat to-upper "joe " .concat "doe"
; returns: "Hi JOE DOE"

"hi " .concat to-upper "joe " |concat "doe"
; returns: "Hi JOE doe"

"Fizz" .print .concat "Buzz" .print |print
; prints:
; Fizz
; Buzz
; FizzBuzz
```

## First argument choice

Because first argument comes from previous expressions on the left, functions in Rye use the _active/input_ value as the first argument if possible. 

It's not always strictly determinable, but in general
in a function one value is the input, the active value and other values are often parameters / settings / options. This sometimes means the same order as we're already used to from other languages, 
but sometimes order is specific.

Let's look at one such chain:

```lisp
; file hello.txt contains "Hello World"

msg: "hello"

read to-file msg .concat ".txt"
|replace "World" "Mars"
|split " "
|second |printv "Planet {}"
; prints: Planet Mars
|write %planet.txt
; writes "Mars" to file planet.txt
```

From example above, in replace we expect active value to be the first argument, but printv and write would probably take template string or filename as the first argument in other languages.


## Second argument from the left

In practice it shows it's sometimes very beneficial if you can take second argument from the left, not the first one. Be it that the value that is coming down the pipe is the second argument, or you want to 
change the order of arguments to get the desired effect. Or with generic functions, more on that later, they dispatch on the _kind_ of first argument, and sometimes active value is second argument.

So if an op or pipe-word ends with a **star** "*" at the end it takes the second argument from the left, not the first.

```clojure
; let's say that the word to replace comes down the pipe, not input string 

word: "apple"

str: "did snake eat the apple?"

word .replace* str "****" |print
; prints: did snake eat the ****?

; or we want a reverse effect

word .concat* "- " |print 
; prints: - apple
```
