---
title: "Rye values and assignment"
date: 2019-02-11T19:30:08+10:00
draft: false
group: true
weight: 100
summary: ""
section: "Aba"
mygroup: true
---

## Rye value types

Rye knows many types of values. From numbers, strings, emails, URI-s, words, special words types, blocks of values ...

```clojure
33
"Hello word"
jim@example.com
https://example.com
some-word
'print
data:
{ "Jane" "Jim" }
```

## Rye code = Rye values

Rye code is a block of Rye values. A block is also a Rye value.

```clojure
print 11 + 22
print "Hello world"
data: { "Jane" "Jim" }
print first data
// prints:
// 33
// Hello world
// Jane
```

```clojure
do {
  print 11 + 22
  print "Hello world"
  data: { "Jane" "Jim" }
  print first data
}
// prints the same as above
```
## Assignment

Words can be linked to Rye values. Set-words are used to create the assignment.

```clojure
a-set-word: "takes value from the expression on the right"
name: "Jane"
age: 33
print name
new-age: print age + 1
// prints:
// Jane
// 34
```

----

### BONUS: Inline set-words

Everything in Rye is also an expression, returns a value. Setwords also return the assigned value, so they can be used inline.

```clojure
prn "All:"
print all-fruits: 100 + apples: 12 + 21
prn { "Apples:" apples }
// prints:
// All: 133
// Apples: 33
```


### BONUS: Left and right set-words

They will make more sense later, but Rye also has left leaning set-words.

```clojure
"Jim" :name
12 + 21 :apples + 100 :all-fruits
```
