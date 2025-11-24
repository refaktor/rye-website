---
title: 'Conventions and patterns'
date: 2024-02-11T19:30:08+10:00
draft: false
weight: 1000
summary: "some things are not enforced or enforceable, but can bring value just the same"
mygroup: true
---

# Naming conventions

A convention is an unwritten rule not enforced by the language itself.

Well thought out conventions can help programmer (or reader of code) to get additional information and predict code behavior better by looking at code, even
if (s)he doesn't know the particular word definitions. Conventions have a benefit, but also has some mental cost, so we should be very reserved about adding them.

<!--Conventions are a tool, an optimization, that also has a cost. They are very helpful, but they should be kept to a **handful** few. Everybody can have around
5 conventions in their mind, and these make the code clearer with less verbiage, but if the convention list grows above 10 for example, cost of keeping and
remembering conventions becomes to great. Then it's better to just be explicit. So we have them, but they should seriously be kept at bay. -->

1) **'?' at the end of the word means 'get-'.** Usually the word is a noun, a property, so instead of `get-height get-length get-color` we have `height? length? color?` functions
2) **'!' at the end of the word means that the function changes values in place.** If the word is a noun, it's usualy a property like `account .balance! 120`, but it can also be a verb for example `account .deposit! 30`
3) **functions that return boolean values start with 'is-'.** So we don't have `empty empty?` but `is-empty`
4) **returning functions start with '^'.** Returning functions are quite specific in behaviour, so we want to make them visible and obvious in code.
5) **Methods (previously called Generic-functions) start with Upper case letter (experimental)**
6) If kinds are defined inside a context, first word of a kind is the name of the context


# Patterns

Some of these patterns emerged with use and experimenting and currently we try to hold to them.


## Urls and Open

Most Url-s Kinds have `Open` method. You can explore that with `lg` function, that if given string returns matching functions from
all kinds available.

```
x> lg "Open"
All matching methods for filter "Open":

Kind: file-uri
  Open [Builtin(1): Opens a file for reading. (io) (file-uri//Open)]
  Open\append [Builtin(1): Opens a file for appending. (io) (file-uri//Open\append)]

Kind: ftp-uri
  Open [Builtin(1): Opens a connection to an FTP server. (io) (ftp-uri//Open)]

Kind: https-uri
  Open [Builtin(1): Opens a HTTPS GET request and returns a reader for the response body. (io) (https-uri//Open)]

Kind: mqtt-uri
  Open [Builtin(1): Opens a connection to an MQTT broker. (mqtt) (mqtt-uri//Open)]

Kind: mysql-uri
  Open [Builtin(1): Opens a connection to a MySQL database. (mysql) (mysql-uri//Open)]
  Open\pwd [Builtin(2): Opens a connection to a MySQL database with separate password parameter. (mysql) (mysql-uri//Open\pwd)]

Kind: sqlite-uri
  Open [Builtin(1): Opens a connection to a SQLite database file. (sqlite) (sqlite-uri//Open)]

...
```

## Function and function variants

`\` in a function name denotes a variant of the function. A variant often takes different (usually additional) arguments than the
base function. `join { "a" "b" }` and `join\with { "a" "b" } ";"`. 

You can see current variants of builtins in Rye console by typing:

```
x> lcp\ "\\"
 ^fix\match [Builtin(2): Error handling switch that matches error codes with handler blocks and sets the return flag. (error-handling) (^fix\match)]
 any\with [Builtin(2): Applies each expression in the block to the provided value until a truthy result is found and returns it. (base) (any\with)]
 append\bytes [Builtin(2): Appends two byte arrays into one. (io) (append\bytes)]
 append\many! [Builtin(2): Appends all values from a block to a collection (block, list or string) in-place and returns the modified collection. (base) (append\many!)]
 change\nth! [Builtin(3): Changes the value at a specific position in a block or list in-place and returns the modified collection. (base) (change\nth!)]
 clone\ [Builtin(2): Creates a copy of a context and evaluates a block of code inside the clone. (base) (clone\)]
 clone\deep [Builtin(1): Creates a deep copy of a context with the same state and parent relationship, recursively copying all nested objects. (base) (clone\deep)]
 context\pure [Builtin(1): Creates a new context using Pure Context (PCtx) as parent, preventing access to regular context changes. (base) (context\pure)]
 decode\base64 [Pure Builtin(1): Decodes a base64-encoded string back to its original form. (base) (decode\base64)]
 defer\ [Builtin(2): Registers a block of code with an injected value to be executed when the current function exits or the program terminates. Works like 'with' but deferred. (base) (defer\)]
 display\custom [Builtin(2): Interactively displays a Table in the terminal with a custom rendering function for each row. (base) (display\custom)]
 do\in [Builtin(2): Takes a Context and a Block. It Does a block in current context but with parent a given Context. (base) (do\in)]
 do\inside [Builtin(2): Takes a Context and a Block. It Does a block inside a given Context. (base) (do\inside)]
 ...
```

## Context co-variants

Context enclose more specific functionlities. They enable short local (lower case) words. For example context `os` can have `ls`, `cd` and `mv` words. But sometimes we want the related but different functionality with the same words. Word `red` in term context provides ansi ascii code sof red font color and in context `html` it would provide a hex code of red color "#ff0000".

But a context co-variant are two context with most of the same words, but providing different functionality based on cotext. Context osv (os visual) has the same functions as context `os`, but osv provides result visually for programmer to see, and `os` provides results as data structure for program to work on. `osv\ls` nicely lists the files in currend directory while `os\ls` returns a bloc of files in the current directory.

This means that you can reuse your knowledge of functions in two different scenarios (contexts).

## Naming symmetries

We should try to use consistent symmetric pairing between functions.

* Load <=> Save
* Read <=> Write
* Open <=> Close


# Internal dilemmas

**When should something in a builtin library be a regular builtin function vs. a method ~~generic function~~?**

Generic functions work with "Kindered?" value types like Native and Uri. An integer is an integer, a string is a string, but there are unlimited number of Native values and also many types of Uri schemas.

So Native and Uri always have a kind because without it its behavior is unspecified. Rye by itself has no functions to deal directly with Natives. Only the Kind of a native determines what functions are available for specific native value.

**Function variant vs. a just another function**

With introduction to function variants many function that would before be phrases like: `load`, `load-csv`,`load-xlsx` make more sense to be variants. `load\csv`,  `load\xlsx` seem to make more sense in this case, as it's the same behavior, similar arguments, but for a slightly different case. All 3 load external data into base Rye values. But we would make a function that loads a page and save it as a file it should not be a variant of `load`. But you see that many multiple word builtin names are just variants. 


For example `parse-json` made sense to me, because we don't have other `parse` function that it would be a variant of. But looking at the `load` variants above and it's base task it makes sense that `parse-json` is a nother variant of `load`, because it does return base Rye values. So we will be renaming it to `load\json`. And `parse-json` would make sense for exampel if we make a function that works with JSON, but doesn't return Rye values, but for example is an event based parser (dialect). 