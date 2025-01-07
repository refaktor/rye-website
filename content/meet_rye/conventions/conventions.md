---
title: 'Naming conventions*'
date: 2024-02-11T19:30:08+10:00
draft: false
weight: 1000
summary: "some things are not enforced or enforcable, but can bring value just the same"
mygroup: true
---


Convention is an unwritten "rule" not enforced by the language itself.

Conventions are a tool, an optimisation, that also has a cost. They are very helpfull, but they should be kept to a **handfull** few. Everybody can have around
5 conventions in their mind, and these make the code clearer with less verbiage, but if the convention list grows above 10 for example, cost of keeping and
remembering conventions becomes to great. Then it's better to just be explicit. So we have them, but they should serriously be kept at bay.

1) **'?' at the end of the word means 'get-'.** Usually the word is a noun, a property, so instead of `get-height get-length get-color` we have `height? length? color?` functions
2) **'!' at the end of the word means that functions changes values in place.** If the word is a noun, it's usualy a property like `account .balance! 120`, but it can also be a verb for example `account .deposit! 30` 
4) **functions that return boolean values start with an 'is-'.** So we don't have `empty empty?` but `is-empty?`
5) **returning functions start with '^'.** Returning functions are quite specific in behaviour, so we want to make them visible and obvious in code.

# Rye concepts

_a work in progress unordered collections of contepts I think we should follow with Rye_ 

* Rye code is not organized in lines or statements, newlines have no meaning and everything is an expression. Rye code is
  formed with scentences. A series of related words and values that define a specific meaning or goal

