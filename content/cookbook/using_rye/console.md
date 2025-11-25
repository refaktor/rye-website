---
title: "Console"
weight: 200
_date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Using Rye console (REPL)"
mygroup: true
---

The Rye console is an interactive environment for evaluating Rye code and exploring the language. It provides specialized functions for navigating contexts, discovering available functions, and understanding the structure of your program's environment.

## Entering the Rye console

When you start Rye with the `rye` command, you enter the interactive console. A new context is created with the builtin functions available as the parent context.

```
$ rye
Welcome to Rye console
x> ; your cursor will be here
```

## Exploring contexts

The Rye environment is organized around **contexts** - containers for words (variables and functions). You can navigate and explore these contexts using console-specific functions.

### List current context

Use `lc` (list context) to see what's defined in your current context:

```
x> lc
Empty context
[Bool: true]    ; the returned value
```

### List parent context

Use `lcp` (list context parent) to see the builtin functions available:

```
x> lcp
Context: Context of builtins
 ^check: [BFunction(2): Returning Check. (core) (^check)]
 ^fail: [BFunction(1): Returning Fail. (core) (^fail)]
 ^fix: [BFunction(2): Fix as a returning function. If Arg 1 is failure, do the block and return to caller. (core) (^fix)]
 _+: [Pure BFunction(2): Adds or joins two values together (Integers, Strings, Uri-s and Blocks) (core) (_+)]
 _-: [Pure BFunction(2): Subtracts two numbers. (core) (_-)]
 _*: [Pure BFunction(2): Multiplies two numbers. (core) (_*)]
 ...and hundreds more...
```

### Filtering context listings

Use `lc\` with a string to filter functions by name:

```
x> lc\ "print"
Context: Context of builtins
 print: [BFunction(1): Prints a value and adds a newline. (core) (print)]
 print\csv: [BFunction(1): Prints CSV formatted output. (core) (print\csv)]
 print\json: [BFunction(1): Prints JSON formatted output. (core) (print\json)]
 printv: [BFunction(2): Prints a value with custom formatting. (core) (printv)]
```

Filter by value type using a word:

```
x> lc\ 'context
Context: Context of builtins
 math: [Context: Mathematical functions]
 os: [Context: Operating system functions]
 pipes: [Context: Pipeline functions]
```

## Navigating contexts

### Change to a context

Use `cc` (change context) to move into a specific context:

```
x> cc math
x> lc
Context: Mathematical context
 sin: [BFunction(1): Sine function]
 cos: [BFunction(1): Cosine function]
 ...
```

### Change to context parent

Use `ccp` (change to context parent) to move up one level:

```
x> ccp
; Returns the context you're moving from
x> lc
; Now shows the parent context
```

### Change to previous context

Use `ccb` (change context back) to move back to the previous context. Runtime stores a stack of contexts you navigated through:

```
x> ccp
; Returns the context you're moving from
x> lc
; Now shows the parent context
```

## Generic functions

Rye supports **generic functions** that dispatch based on the type of their first argument. These are listed separately from regular functions.

### Listing kinds

Use `lk` (list kinds) to list available kinds.

```
x> lk
 Ed25519-priv-key
 Ed25519-pub-key
 Gin-context
 Gin-group
 Gin-router
 Go-server
 Go-server-request
 ...

x> lk\ "uri"
 file-uri
 ftp-uri
 http-uri
 https-uri
 mqtt-uri
 mysql-uri
 ...
```


### List generic functions

Use `lg` (list generic) to see generic functions that work with specific types:

```
x> lg 'https-uri
Methods (https-uri):
 Get [Builtin(1): Makes a HTTPS GET request and returns the response body as a string. (io) (https-uri//Get)]
 Open [Builtin(1): Opens a HTTPS GET request and returns a reader for the response body. (io) (https-uri//Open)]
 Post [Builtin(3): Makes a HTTPS POST request and returns the response body as a string. (io) (https-uri//Post)]
 Request [Builtin(3): Creates a new HTTPS request object. (io) (https-uri//Request)]
```

## Console keyboard shortcuts

The Rye console supports various keyboard shortcuts for efficient editing and navigation:

### Tab completion

This is still in development and fine tuning.

Currently, if you press Tab without any text entered console will display words in current context and 
you will be able to cycle over them and see their values.

If you already antered text and then press tab console will show suggestions based on all indexed words.
You will again be able to cycle over them and see their values.

```
x> <Tab>
; [name] say-hi
; [String: Anne]

: prin<Tab>
; [print] print2 print\csv print\ssv
```

### Control key commands

- **Ctrl+A**: Move cursor to beginning of line
- **Ctrl+E**: Move cursor to end of line  
- **Ctrl+B**: Move cursor backward (left) one character
- **Ctrl+F**: Move cursor forward (right) one character
- **Ctrl+D**: Delete character under cursor (or exit console if line is empty)
- **Ctrl+K**: Delete from cursor to end of line
- **Ctrl+U**: Delete from cursor to beginning of line
- **Ctrl+W**: Delete word before cursor
- **Ctrl+L**: Clear screen
- **Ctrl+C**: Interrupt current input and start fresh prompt

### Multi-line input

If blocks or strings don't finish by the end of the line
console will go into multiline mode and you will be able to write
multiple lines until you close the block or string.

```
x> list {
... 1 2 3
... 4 5 6
}
[List: [1 2 3 4 5 6]]
```

## Tips for effective console use

- Use **Tab completion** to discover available functions and avoid typos
- Use `lc\` or `lcp\` with search terms to quickly find functions related to your task
- Use `lk` or `lg` to discover generic functions that work with specific kinds of values
- The console maintains command history - use **up arrow / down arrow** to recall previous commands
- Use `enter-console "description"` within your programs to drop into console
- Use **Ctrl+L** to clear the screen when output gets cluttered

The Rye console is designed for exploration and experimentation - don't hesitate to try functions and inspect the environment to learn how the language works!
