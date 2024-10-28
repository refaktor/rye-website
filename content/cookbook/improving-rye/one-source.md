---
title: "One source of truth*"
date: 2024-10-24T19:30:08+10:00
draft: false
group: true
weight: 150
summary: "Basic building blocks of Rye are builtin functions. To have them tested, documented is work and unifying the position."
mygroup: true
---

*work in progress*

# Builtins

Builtin functions are the main building blocks of Rye, the only active component.

# Runtime information

Builtin functions and regular function have some information avaliable at runtime. These are provided in Go or Rye code directly.

* name
* number of arguments
* kind to dispatch to - in case it's a generic function
* docstring

# Loadable information

We don't want to make runtime too heavy, so additional infomation will be available as an offline structure that the runtime can use to display information in console for example. Or for generating information on the web.

For every builtin this consists of:

* doc (optional longer freeform description of function)
* tests (using the tests/ format) - each builtin should have at least 1 test for now
* examples (optional - if not there first 3 tests are also considered examples) - examples have names and can be referenced
* argument names - can be provided or are parsed from Go code
* argument types - are parsed from Go code
* tags (optional)

# Providing information

Runtime information is provided as part of making a builtin function, which has these fields available.

Loadable information is provided as a comment above the builtin function definition in a Go file. Example of such
comment:

```
	// The longer description of function, where it adds value, 
	// but if dostcstring is enough, don't write it just for the 
	// sake of it.
	// Args:
	//  * subtrahend - the value being substraced from
	//  * minuend
	// Tests:
	//  equal { 10 - 2 } 8
	//  equal { _- 2 10 } -8
	// Example: Subtracting 3 values
	//  ; be careful how op-words (like -) work
	//  print ( 100 - 10 ) - 1
	//  ; prints 89
	//  print 100 - 10 - 1
	//  ; prints 91
	// Examples:
	//  * Name of another example
	// Tags: #math #core
	"_-": { // **
		Argsn: 2,
		Doc:   "Subtracts two numbers.",
		Pure:  true,
		Fn: func(ps *env.ProgramState, arg0 env.Object, arg1 env.Object, arg2 env.Object, arg3 env.Object, arg4 env.Object) env.Object {
	... 
	... (Go definition of builtin)
```
