---
title: 'Rye evaluator'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 200
summary: "The rules that Rye language evaluator works by."
mygroup: true
---

We have seen in the previous chapter that Blocks of Rye values can be used to represent many different things. [REBOL](http://rebol.com) showed how they can 
also represent a programming language.

## A programming language

<!--REBOL proved that they can also represent a programming language. 
Now imagine we use this data format as a programming language. We have a uniform format. We also want clear and consistent rules of evaluation.
Credit for both of these concepts comes from a language called REBOL.-->

We want to have simple, uniform and consistent rules of evaluation. These are the rules of Rye1 evaluator:

![Rye code scheme](../rye_code_scheme_3.svg)
	

## Evaluation rules in action

Let's start a Rye shell and see the evaluation rules from the scheme above in action.

### Direct values

Direct values evaluate to themselves:

![Direct values](../direct_values_2.png)

### Words

Words are usually bound to a value. Value can be a direct value or a function. Direct values evaluate to themselves, functions take arguments, if needed, run and return the result or failure.

![Words](../words_2.png)

### Special word types

For now we need just 3 special word types.

**Lit-word** (literal word) evaluates to the word itself. Word is an indexed entity and can be used in similar way as enums would be.

**Set-words** bind values to words. Code on the right of them get's evaluated, if needed, to get the value. The value is returned from such expression.

**Get-words** always gets the value they are bound to. We need them because if word is bound to a function, with normal word we run the function and get the result, but sometimes we need the function itself.

![Special words types](../special_words_2.png)

### Factorial example

Let's use the evaluation rules we just learned and write a recursive factorial function.

![Factorial](../factorial_3.png)

So what are the parts that make the example work? We already saw, by using get-words in the shell, that **is-equal**, **mul** and **dec** are functions and what they do.

**fac** is a word we bind our factorial function to. 

**fn** is a function that creates _functions_ (yeah really!). It takes two arguments, both blocks. First is the argument list, second it the body of the function.

**either** is a function that works like if-else statement in other languages. It takes 3 arguments, a boolean and two blocks (of code), one for true, another for false case. Either returns the last value of the block it runs.

![fn either](../fn_either.png)

**BFunction** means a Builtin Function, a function defined in Rye's host language Go, versus a Function defined in Rye itself.

Because Rye interpreter doesn't evaluate blocks on it's own, it doesn't need any special forms for control flow, for function definition, etc ... so it also doesn't need keywords or macros. All these things are ordinary functions. Meaning you can have more of them, you can make your own, you can load them or not load them on a "library" level.
