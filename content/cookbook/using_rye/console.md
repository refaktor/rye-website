---
title: "Console*"
weight: 200
---

You can use Rye console, to evaluate Rye code. But Rye has some specific functions that usually aren't used
in normal Rye programming, but can be very handy in the console. For one thing, the console helps you explore
the Rye environment.

Rye environment is formed by Words and words come in Contexts. You can navigate, list, use and modify contexts
and assign words.

## Entering Rye console

When you enter a console in a usual way by calling `rye` for example, a new Context is created and the parent context
of it is the context where all the Builtins are defined. 

## List context

You can use function `lc` (list context) to see the current
empty context.

```
x> lc
Empty context
[Bool: true]    ; the returned value
```

Empty context is not that interesting, but let's check out the parent context.

## List context parent

Rye has function `lcp` (list context parent). If you call this function in your console a long list of words, mostly bound t
BFunction-s (builtin functions) will appear.

```
x> lcp
Context:
 ^check: [BFunction(2): Returning Check. (core) (^check)]
 ^fail: [BFunction(1): Returning Fail. (core) (^fail)]
 ^fix: [BFunction(2): Fix as a returning function. If Arg 1 is failure, do the block and return to caller. (core) (^fix)]
 ^if: [Pure BFunction(2): Basic conditional with a Returning mechanism when true. Takes a condition and a block of code. (core) (^if)]
 ^otherwise: [Pure BFunction(2): Basic conditional with a Returning mechanism when true. Takes a condition and a block of code. (core) (^otherwise)]
 ^require: [BFunction(2): Returning Require. (core) (^require)]
 ^tidy-switch: [BFunction(2):  (core) (^tidy-switch)]
 _*: [Pure BFunction(2): Multiplies two numbers. (core) (_*)]
 _+: [Pure BFunction(2): Adds or joins two values together (Integers, Strings, Uri-s and Blocks) (core) (_+)]
 _-: [Pure BFunction(2): Subtracts two numbers. (core) (_-)]
 ...
 etc.
```

## Change context

In fact you can move into contexts in similar (but not the same) way as you move into directories on your hard drive. Function `cc ctx-name` changes / moves
you to that context. But since we are in an empty context let's first move to a parent context using `ccp` (change context to parent). If you then use `lc` command
you will see the difference.

```
x> ccp
; Returns and so displays the context we are moving from

x> lc
Context:
 ^check: [BFunction(2): Returning Check. (core) (^check)]
 ^fail: [BFunction(1): Returning Fail. (core) (^fail)]
 ^fix: [BFunction(2): Fix as a returning function. If Arg 1 is failure, do the block and return to caller. (core) (^fix)]
 ^if: [Pure BFunction(2): Basic conditional with a Returning mechanism when true. Takes a condition and a block of code. (core) (^if)]
 ...
 etc.
```

## List with a filter

In Rye we have a convention of naming variations of functions in the form of `word\variation`. This is still just a regular word. So another convention says, if
you want to make sort of default variation with some more arguments, you can use just `word\`. This function usually takes 1 more argument thatn function `word`.

So we have a function `ls\` that takes another argument which it filters on the results. If the argument is string it filters over matching words.

```
x> lc\ "print"
Context: Context of builtins
 print: [BFunction(1): Prints a value and adds a newline. (core) (print)]
 print\csv: [BFunction(1): Prints a value and adds a newline. (core) (print\csv)]
 print\json: [BFunction(1): Prints a value and adds a newline. (core) (print\json)]
 print\ssv: [BFunction(1): Prints a value and adds a newline. (core) (print\ssv)]
 printv: [BFunction(2): Prints a value and adds a newline. (core) (printv)]
```

If the argument is a word, the filter is applied over the type of the value. [TODO]

```
x> lc\ 'context
Context: Context of builtins
 math: [Context: ...]
 pipes: [Context: ...]
 os: [Context: ...]
```

## List with a full text search [TODO]

```
x> lc\full "spreadsheet add"
Context: Context of builtins
 add-column!: [BFunction: ...]
 add-column: [BFunction: ...]
 add-row!: [BFunction: ...]
 ...
```

## List generic functions [TODO]

Rye has a concept of generic functions. Functions that dispatch of the kind of the first argument. They live in a differently structured namespace.
To `lg` (list generic) you have to add a kind of the function you are looking for.

```
x> lg 'uri-scheme
Context: Generic context
 open: [BFunction: ...]
 ...
```
&nbsp;

`lg\` accepts the kind of the first argument and a string to filter by.

```
x> lg\ 'rye-pipe "out"
Context: Generic context
 open: [BFunction: ...]
 ...
```

If the first function is a string it does aproximate match on kind also.

```
x> lg "file"
Context: Generic context
 ; lists all generic functions that dispatc on kinds related to files
 ...
```


