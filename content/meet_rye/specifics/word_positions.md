---
title: 'Word positions'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 292
summary: "The four positions a function can take in an expression."
mygroup: true
---

## Four ways to call a function

In Rye, the same function can be called in four different positions:

| Position | Syntax | First argument from |
|----------|--------|---------------------|
| Regular word | `word` | right |
| Dot-word (op-word) | `.word` | left (immediate) |
| Operator | `+` `<word>` | left (immediate) |
| Pipe-word | `\|word` | left (after full evaluation) |

## Example with `add`

```clojure
add 10 20       ; regular word
10 .add 20      ; dot-word
10 + 20         ; operator (op-word by default)
5 + 5 |add 10   ; pipe-word (waits for 5 + 5 first)
```

## Evaluation order

- **Regular words**: collect arguments going right
- **Dot-words/operators**: evaluate right-to-left among themselves
- **Pipe-words**: evaluate left-to-right, wait for left side to complete

```clojure
1 + 2 * 3       ; op-words: 1 + (2 * 3) = 7
1 |+ 2 |* 3     ; pipe-words: ((1 + 2) * 3) = 9
1 + 2 |* 3      ; mixed: (1 + 2) * 3 = 9
```

See [Evaluation priorities](/meet_rye/specifics/evaluation_priorities) for details.
