---
title: 'Op and Pipe-words'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 300
summary: "A different ways to call regular functions that enable left to right code flow."
mygroup: true
---

And we are finally here. The following are the first noticeable things that probably hit you in the face when you looked at various Rye examples.

## Calling functions

Calling functions in Rye is simple. You use a word that is bound to a function, and provide the arguments.

When the evaluator gets to a word bound to a function, it goes forward collecting the necessary number of values for arguments and then evaluates the function. Here are few simple examples:

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

Now we will call the same functions with the same arguments, but we'll use op-words instead of regular words. Op-words begin with a dot. In case of an op-word, the first argument is taken from the left.

```clojure
"Call me" .print
; prints: Call me

100 .inc
; returns: 101
```

As we see in the following examples, only the first argument is taken from the left. The rest follow like regular words.

```clojure
"Fizz" .concat "Buzz"
; returns: "FizzBuzz"

15 .factor-of 3
; returns: 1

"Hello World" .replace "World" "Mars"
; returns: "Hello Mars"
```

## Operators

What are operators then? Operators in Rye are nothing special, they are just regular functions usually taking two arguments. The only difference is that one and two letter operator characters are by default recognized as **op-words**, they don't need the dot in front. 

The regular spelling of the word in this case is with an added underscore in front of the op-word. For the op-word `+`, the regular word is `_+`.

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

Pipe words behave similarly or in simple cases exactly the same as op-words. So you will get the same results in these examples as with op-words:

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

It's not always strictly determinable, but in general in a function one value is the input, the active value and other values are often parameters / settings / options. This sometimes means the same order as we're already used to from other languages, 
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

From the example above, in replace we expect the active value to be the first argument, but printv and write would probably take template string or filename as the first argument in other languages.


## Second argument from the left

In practice, sometimes it's very beneficial if you can take second argument from the left, not the first one. For example, the value that is coming down the pipe is the second argument, or you want to change the order of arguments to get the desired effect. Or with generic functions (more on that later), they dispatch on the _kind_ of first argument, and sometimes the active value is the second argument.

So if an op or pipe-word ends with a **star** "*" at the end, it takes the second argument from the left, not the first.

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

## Warning: math expressions

We saw Operators before, but let's review it again. 
In Rye, operators like `+ - * /` are regular functions in the form of op-words. The only difference is that operators are loaded as op-words by default while regular `.words` need a dot in front to be
op-words. Their non op-word (prefix) form is: `_+ _- _* _/ `, so: 

```clojure
1 + 2 + 3
; return 6

_+ 1 _+ 2 3
; or
_+ _+ 1 2 3
; return 6
```

And you can use the same function as pipe words using: `|_+ |_- |_* |_/` or shorter `|+ |- |* |/`. But what does all this means for functions (or operators) that are **not associative** and **commutative** like addition is?
It means you have to be aware of the fact that operators are still op-words and behave like them, not like math operators.

```clojure
; add 12 and 3 and divide the result by 3 (prefix)
_/ _+ 12 3 3
; returns 5

; using pipe word for division
12 + 3 |/ 3
; returns 5

; using just op-words
12 + 3 / 3
; returns 13 -- Oops!
; op-words evaluate from right to left, pipe-words left to right 

; subtracting 3 numbers also doesn't do what you would expect from math
100 - 12 - 5
; returns 93
; it first subtracted 12 - 5 and got 7 and it subtracted 7 from 100

; addition and multiplicaton have same operator precedence
; but if this was a math expression, we would expect the result 56
10 * 5 + 2 * 3
; returns 110
; from left to right it does:
; 2 * 3 = 6
; 6 + 5 = 11
; 11 * 10 = 110
```

Why is that? Rye comes from the family of languages where the whole language is uniform and math expressions are not treated any differently. These languages usually see it as a plus, that you don't have to learn (complex?)
operator precedence rules. Yes, we all know precedence rules between `+ - * /` but what about all other operators or functions like `! and or & > < ...`, so they might have a point.

In REBOL, math expressions are always evaluated left to right, which is more natural than right to left, but you will get shot in the foot just the same if you are not aware of it. Lisp goes a step beyond and doesn't even
have infix operators or functions. In Forth it's the same, all functions (also operators) are postfix.

```clojure
; in REBOL
10 * 5 + 2 * 3
; returns 156 (left to right)

; in Lisp this would be written, which has it's estetics too
(+ (* 10 5) (* 2 3))

; and in Forth (Factor)
10 5 * 2 3 * +
```

All 3 languages value internal consistency above conventions and so does Rye. So what can you do in Rye in this case:

```clojure
; in Rye (and REBOL) we can use parenthesis to explicitly define the order
; of operations we want, and this is IMO the most natural way to do this

( 10 * 5 ) + ( 2 * 3 )
; or just
( 10 * 5 ) + 2 * 3
; return 56

; technically we could use the distinction between op and pipe words
10 * 5 |+ 2 * 3
; returns 56

; and Rye has a math dialect, which follows math precedence rules
; it's defined in a math context, where you can find plenty of math
; specific functions (do `cc math lc` to see them)

math/calc { 10 * 5 + 2 * 3 }
; returns 56
```

