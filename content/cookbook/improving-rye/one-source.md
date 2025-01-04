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

Builtin functions are the main building blocks of Rye - the only active component. And there is a lot of them. The base builtins file currently has 283 builtin functions. 
I believe just syncronizing / looking-up / ordering / finding missing builtins' code, tests, examples, and other snippets of information has a big cost. 
But this cost is removed if all this is in one place. 

You see the code for a builtin, you see the docstring, the argument list, unit tests. If you change the code, you can right there update the tests and arguments. If test fails
you can right there fix the code for it to work.

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

# Rbit tool

Still work in progress but you can find this tool in cmd/rbit. If not built yet call the local `./build`. Then you can run it to get the help information:

## Generate info file

Info file holds (will hold) all the information there is about each builtin. Information will be available from Rye console and it will be used to generate
function reference on the web. Info files are always generated / regenerated, so manual changes will be lost.

Rbit's help is quite self explanatory.

```
(mycomp) -> ./rbit
╭────────────────────────────────────────────────────────────────────────────────────────────---
│ rbit - rye builtin info tool - https://ryelang.org
╰───────────────────────────────────────────────────────────────────────────────────────---

 Usage: parse [options] [filename]
  -help
    	Displays this help message.
  -ls
    	List builtins files
  -stats
    	Show stats about builtins file
  rbit                                                       # shows this help
  rbit ../../evaldo/builtins.go > ../../info/base.info.rye   # generates the info file
  rbit -stats ../../evaldo/builtins.go                       # gets builtins file stats
  rbit -ls ../../evaldo/                                     # lists builtin files
  rbit -help                                                 # shows this help

 Thank you for trying out Rye ...

```

## Rbit stats

Call it with a `-stats` flag and a builtins file and you will get some basic statistics of where you are with this file. The output is something you can
look at, but you can also load it as a regular Rye code. Info files go into `info/` folder and there are further tools to work with them.

```
(mycomp) -> ./rbit -stats ../../evaldo/builtins.go 
stats {

	functions       	283
	tested-functions	57
	tests           	131
	examples        	0

	test-coverage   	20.1%
	tests-per-func  	2.3
}
```

## Generating info file

As we've seen in the Rbit's help above, this is how you generate an info file. There is one info file per one `evaldo/builtins*` Go file.

```
# generates the info file
rbit ../../evaldo/builtins.go > ../../info/base.info.rye
```

&nbsp;

# Running tests

You can enter the `info/` folder and since there is a `main.rye` you can run it with a dot and you will get the following:

```
(mycomp) -> cd info/
(mycomp) -> rye .
# Rye's simple testing tool
	
 use test or doc command

Examples: 
 rye . test           # runs all tests
 rye . doc            # generates the docs out of tests
 rye . test ls        # lists the test sections available
 rye . test basics    # runs a specific test section
```
&nbsp;

So you can use `-ls` to list files or run a specific file. `demo` is a very short one, main builtins are in `base`.

```
(mycomp) -> rye . test demo

# DEMO #

demo
 just testing the testing

 demo1 ✓ 
 demo2 ✓ 
```
&nbsp;

There is also a `./regen` script in the same folder, that will regenerate all info files. The CLI interfaces will get unified between these tools.

&nbsp;

# Generating HTML Reference

_TODO_
