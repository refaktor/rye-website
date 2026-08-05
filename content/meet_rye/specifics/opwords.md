---
title: 'Pins, Bridges and Walls'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 292
summary: "Three word types that take their first argument from the left: dot-words (pins), op-words (bridges), and pipe-words (walls)."
mygroup: true
---

Rye's base dialect has no special forms — all code evaluates the same way, based only on the types of words in a block. Three word types take their first argument from the **left**, enabling left-to-right code flow, each with a different behavior. Think of them as physical elements:

- **Pins** (dot-words) attach tightly to what's immediately beside them
- **Bridges** (op-words) connect two peers
- **Walls** (pipe-words) block everything until it's fully evaluated

## Four ways to call a function

The same function can be called in four positions:

| Position | Syntax | First argument from | Associativity |
|----------|--------|---------------------|---------------|
| Regular word (front) | `word` | right | - |
| Dot-word (pin) | `.word` | left (immediate) | left-to-right |
| Op-word (bridge) | `+` or `<word>` | left | left-to-right |
| Pipe-word (wall) | `\|word` | left (after full eval) | left-to-right |

```clojure
add: ?_+        ; assign a function bound to _+ to add word

add 10 20       ; regular word
10 .add 20      ; dot-word (pins to 10)
10 + 20         ; operator (op-word, bridges 10 and 20)
10 <add> 20     ; op-word with angle brackets
5 + 5 |add 10   ; pipe-word (waits for 5 + 5 first)
```

## Dot-words: The Pin

Dot-words pin to the first value on the left and **create atoms** — tightly bound units that complete before other operations can interfere.

```clojure
inc 100        ; regular word - argument from the right
; returns: 101

100 .inc       ; dot-word - pins to 100
; returns: 101
```

### Chaining dot-words: left-to-right

When dot-words are chained, they evaluate **left-to-right** — each forms its atom and passes the result to the next:

```clojure
5 .+ 3 .* 2
; first atom: 5 .+ 3 = 8
; second atom: 8 .* 2 = 16
; returns: 16

10 .inc .inc .string
; 10 .inc = 11, 11 .inc = 12, 12 .string = "12"

"hello world" .upper .length?
; "HELLO WORLD" .length? = 11
```

### Atoms don't leak

When a pin collects its arguments, it **doesn't consume other pins or bridges**:

```clojure
10 .+ 3 .string     ; PIN grabs 3, not .string → atom: 13 → "13"
10 .* 2 + 5         ; PIN grabs 2, leaves + 5   → atom: 20 → 25
10 - 10 .dec        ; .dec pins to right 10 = 9  → 10 - 9 = 1
```

### With multiple arguments

Only the first argument comes from the left. Additional arguments are collected normally:

```clojure
"Hello World" .replace "World" "Mars"
; returns: "Hello Mars"

10 .+ 5 .string
; .+ takes 10 from left, 5 from right → atom: 15 → "15"
```

## Op-words: The Bridge

Op-words create bridges between two expressions. Unlike pins, bridges **allow atoms to form** in their arguments.

All operators like `+ - * /` are op-words and evaluate **left-to-right**:

```clojure
12 - 6 - 4
; (12 - 6) - 4 = 2

10 * 5 + 2 * 3
; ((10 * 5) + 2) * 3 = 156
```

The prefix form of an operator uses an underscore: for `+`, the regular word is `_+`.

### Bridges allow atoms inside

```clojure
10 - 10 .dec
; right atom: 10 .dec = 9 → bridge: 10 - 9 = 1

10 .inc - 10 .dec
; left atom: 11, right atom: 9 → bridge: 2
```

### Angle brackets

Any word can be turned into an op-word with angle brackets `< >`:

```clojure
"Hello" <concat> "World"    ; returns: "HelloWorld"
{ "Jim" "Jane" } <join\with> ", "  ; returns: "Jim, Jane"
```

## Pipe-words: The Wall

Pipe-words act as a **wall** — everything on the left must fully evaluate before the pipe-word takes the result.

```clojure
inc 10 * 10     ; * bridges 10 and 10: inc 100 = 101
inc 10 |* 10    ; wall: inc 10 = 11, then 11 * 10 = 110
```

### Building pipelines

```clojure
read %data.txt
|parse-json
|filter { .active }
|length?
|print
```

Each `|` ensures the previous step completes before continuing.

## Combining All Three

```clojure
; Pins create atoms, bridge connects, wall separates stages
data .filter { .age > 18 }
|map { .name .upper }
|join\with ", "
|print
```

Debugging with `.print` vs `|print`:

```clojure
10 + 3 .print * 100 |print / 3
; .print pins to 3 → prints: 3
; |print waits for whole left → prints: 1300
; returns: 433.333...
```

## Priority summary

| Type | Syntax | Associativity | Captures |
|------|--------|---------------|----------|
| Pin (`.word`) | `.word` | left-to-right | immediate left value + own args |
| Bridge (`<word>`, `+`) | `<word>` or operator | left-to-right | one peer on each side |
| Wall (`\|word`) | `\|word` | left-to-right | full left evaluation |
| Set-word | `x::` | right | everything to the right |
| Left set-word | `::x` | left | like pipe-word |

**Set-words and mod-words** have even higher priority than pipe-words — they capture everything that follows (or precedes) to the end of the expression:

```clojure
x:: 12 - 6 - 4       ; x = 2
x:: inc 10 |* 10      ; x = 110

12 - 6 - 4 ::x        ; x = 2
101 ::x + 1 |+ 2 |* 3 ::y
; x = 101, y = 312
```

When in doubt, parentheses give explicit control:

```clojure
( 12 - 6 ) - 4   ; result: 2
12 - ( 6 - 4 )   ; result: 10
```

## Second Argument from the Left

Sometimes you need the **second** argument from the left. Add a star `*` at the end:

```clojure
word: "apple"
str: "did snake eat the apple?"

word .replace* str "****" |print
; prints: did snake eat the ****?
```

This works for all three types: `.word*`, `<word>*`, `|word*`.
