---
title: 'First class Contexts'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 900
summary: "Context or evaluation scope is a first class value in Rye and Context is king."
mygroup: true
---

_"Context is king, I shouted, then ran from the king's guard."_
 

Although Rye's contexts are simple structures, we have multiple functions for creating them and it seems even more uses for them. Naming of functions tries to be intent based, but we're still exploring the area so it can change.

**If you came directly to this page, maybe first have a look at [context basics](../../basics/context/).**

To recap: Context is used as **scope** for Rye words. It's also just another **value type** in Rye. Structurally, context is a dictionary of words and values, an optional link to the parent context and an optional docstring.

## Basic context

The  most common way to create or just use the features of a context is by function `context`. Function evaluates a block of code in it's newly created context and returns the context itself. It sets contexts parent to the *current-context* when created.

```clojure
pos-printer: context { 
  codepage: "latin2" 
  out: fn { x } { prn x } 
  crlf: does { print "" } 
  info: does { out "Cp: " + codepage , crlf } 
}
```

## Context path

We can use a **context path**, another literal value type in Rye, to address values inside a context. Evaluator uses _cpath_ in the same manner as it uses words. If a word in a context is bound to a function it evaluates it, if not it returns the value bound to it.

```clojure
print pos-printer/codepage
; prints: w1250

pos-printer/info
; prints: Cp: w1250

pos-printer/out "Hello world"
; prints: Hello world (no newline)

pos-printer/crlf
; prints newline
```

### No direct changes

We can't use the equivalent of **set** or **mod-words**, to change the values inside a context. If you come from Rebol, this is possible there, but Rye tries to keep a much tighter control on modification in general. 
Rye only allows direct changes in the current context, but not in sub-contexts and not in the parent contexts. You can only _send messages_ i.e. make function calls there.


```clojure
pos-printer/name: "POS printer"
; Loader error - this syntax doesn't exist

pos-printer/codepage:: "utf-8"
; Loader error - this syntax doesn't exist
```

## Evaluation

### In a context

Besides calling into a context with cpath, we can also just evaluate our code in a context. 

**Do** is a general Rye function that *does* the Rye dialect. A variation `do\in` exists that _does_ code in a specific context. 
Well, not directly inside a context, since we believe things are better kept
separate, not mashed together. Code is still evaluated in it's current context (like with _do_) but with specified context being a parent.

```clojure
do\in pos-printer { 
  print codepage 
  loop 3 {  out "Hello" , crlf } 
  info 
}
; prints:
;  latin2
;  Hello
;  Hello
;  Hello
;  Cp: latin2
```

So how does this work? Evaluator finds values of **out**, **clrf**, **info** and **codepage** in it's now parent context, the context _pos-printer_ that we created above. It evaluates the first 3 because they are functions and retrieves the last. 

But it found _print_ and _loop_ in the parent context of the parent context, where all the built-ins are defined.

<!-- With **do\in** we have access to words in current context and the words defined in a specified context (pos-printer) through the parent link. --> 
We can use functions **lc** (list context) and **lcp** (list context parent) to demonstrate this.

```clojure
name: "Jim"

do\in pos-printer { out name }

do\in pos-printer { ls }  ; list current context
; lists *name* and *pos-printer* ...

do\in pos-printer { lsp } ; list parent
; lists codepage, crlf, out and info 
```

<!-- This is the default way, because we don't want to be able to change the context and we can't directly change anything outside our current context. -->

### Builtin contexts

Rye also comes with some built-in contexts. To ensure we can use short math related words to do math operations **math** built-in functions are defined in their own context.

We can use words from such contexts using cpaths (math/pi, math/sin) like in the next example.

```clojure
deg->rad: fn { d } { * math/pi / 180 }

for { 0 90 180 270 } { .deg->rad .math/sin .print }
; prints:
;  0.00000
;  1.00000
;  0.00000
;  -1.00000
```

Or, if we use a lot of words from math context we rather just evaluate our code with **do\in**:

```clojure
pipe-radius: 3.0919

do\in math { pi * pipe-radius .sq |round\to 2 } :area
; returns: [Decimal: 30.030000]
```


## More ways to create contexts

### Extends function

Function **extends** accepts another context in addition to a block of code and sets the accepted context as a parent to a newly created one. 

This is a general mechanism for creating contexts with specific parents, but it also behaves sort of like class\object inheritance. But Rye is not an OO language, so this is not a pattern around which we usually structure code.

```clojure

animal: context { eat: does { print "I eat" } }

dog: extends animal { voice: does { print "Woof!" } }

cat: extends animal { voice: does { print "Meow" } }

do\in dog { eat , voice }
; prints: 
;   I eat
;   Woof!

cat/voice
; prints: Meow
```

Rye is context oriented language, where words get meaning in specific contexts, and we can construct contexts in a way that makes the most sense. In code below context `circle` isn't viewed as a subclass of math, but the programmer decided 
that it makes sense to define context circle with context math as the first parent.

```clojure
rye .needs { math } ; check if we have math module built-in

circle: extends math { 
	circuference: fn { r } { 2 * pi * r } 
	area: fn { r } { pi * r .sq } 
}

print circle/area 10
; prints: 314.159265

do\in circle { print circuference 10 }
; prints: 62.831853
```

This is a way of giving our code access to words defined in math context, and giving these words a priority for this particular code. 

### Private function


Function **private** uses a context for a completely different reason. It creates a new context and executes code in it, but just returns the **last value**, not the context.

This is useful if we need the result of some operations, but we don't want the intermediate words/variables to pollute our working context.


```clojure
private { 
  12 * 5 :a 
  a - 12 :b 
  a + b 
} :result

print result
; prints 108

ls
; Context: 
;  result: [Integer: 108]
```

<!-- Rye code isn't organised in lines and isn't in statements, but more something like sentences. And sentences usually have one main goal. -->


### Isolate function

We finally arrived at the function that I'm the most excited about. **Isolate** creates a context and during the construction of it, the current context is the parent. But the resulting context then has no parent so it's isolated from everything.

Code evaluated in a resulting context can call **only the functions** that are defined in this isolated context and nothing else.

**This wouldn't be that exciting, but you have to remember that every active component, also all language constructs like `if loop fn ...` in Rye are just functions.**

Imagine having a framework, but you provide a specially crafted context, for the framework programmer to use. Or a distributed computing platform where code is sent between processes, but code is  evaluated in such specialized, safe, isolated contexts. 
So your peer or a client isn't given just endpoints, but a dialect that is composable and can have the full power of a language, or just a very limited subset of it.

Let's create an isolate that represents an ok-printer. It exposes only two of Rye's built-in functions, **print** and **prn** as **pr** and **p**.

```clojure
ok-printer: isolate { pr: ?print p: ?prn }

do\in ok-printer { p "*" p "*" p "*" pr "" pr "OK." }
; prints:
;  ***
;  OK.

; it shouldn't have access to anything else in Rye
do\in ok-printer { print "OK?" }
; Error: word 'print' not found

do\in ok-printer { if 1 { pr "ER!" } }
; Error: word 'if' not found

msg: "WT?"
do\in ok-printer { pr msg }
; Error: word 'msg' not found
```

Now we will add our context the ability to do basic looping. So our dialect can avoid some repetition.

```clojure
ok-printer:: isolate { pr: ?print p: ?prn lo: ?loop }

do\in ok-printer { lo 3 { p "*" } pr "" pr "OK." }
; prints:
;  ***
;  OK.
```

We want to add function **nl**, to avoid printing an empty string for a newline. This is not a direct built-in, but our custom Rye function. This function uses just words from
isolated context itself, so ordinary **fn** or **does** works.

```clojure
ok-printer:: isolate { pr: ?print p: ?prn lo: ?loop nl: does { pr "" } }

do\in ok-printer { lo 3 { p "*" } nl pr "ER!" }
; prints:
;  ***
;  ER!
```

Now lets give our remote programmer that uses the context (for example) the ability to create her own functions.

```clojure
ok-printer:: isolate { 
  pr: ?print 
  p: ?prn 
  lo: ?loop 
  f: ?fn 
  nl: does { pr "" } 
}

do\in ok-printer { ln: f { c } { lo 3 { p c } } , ln "-" nl pr "QL!" }
; prints:
;  ---
;  QL!
```

Now we wish to expose a function, but contrary to **nl** above, this function will use Rye's regular functions that aren't available inside a context in it's body.
Well that is not a problem. We already saw rye can create functions that are executed in specific contexts, so we use that mechanism.


```clojure
ok-printer:: isolate { 
  pr: ?print 
  p: ?prn 
  nl: does { pr "" } 
  ; emphasize 
  em: fn\in { m } parent { 
    .print , 
    produce length? m "" { .concat tail m 1 } \print 
  } 
}

do\in ok-printer { 
  em "CU/" em "BY!" 
}
; prints:
;  CU/
;  ///
;  BY!
;  !!!
```

## Functions in specific contexts

### Fn\in

Similar to **do\in** you can also define a function that is evaluated _in_ a specific context. You create these function with **fn\in**. The body of the function is evaluated in it's own context, like a normal function, but
with parent set to the given context.

<!--Rye has a functions **fn\in** and **fn\inside** which besides arguments list also accept a context. With it you can define a custom context in which you want function to be executed, but there are two
options. **fn\in** executes code of a function in a context where the given context is a parent context, with **fn\inside** the code of the function is executed directly inside the given context. 

**fn\in** is usually enough while \inside option can be used in more specific cases.-->

```clojure
me: context { name: "Jim" }

introduce-to: fn\in { to } me { print [ "Hi" to ", I'm" name ] }

introduce-to "Jane"
; prints: Hi Jane , I'm Jim

introduce-to "Bob"
; prints: Hi Bob , I'm Jim
```

Or if we use the context _ok-printer_ we defined above


```clojure
print-report: fn\in { title msg } ok-printer { em title , pr msg }

print-report "BRKNG!" "all-ok"
; prints:
;  BRKNG!
;  !!!!!!
;  all-ok
```

### Closures

And where do closures fit into this? A closure in Rye is just a specific use of the function **fn\in** with the current context. We do have a helper
function `closure`, but we get the same result if we do:

```clojure
make-adder: fn { a } { fn\in { b } current { a + b } }

add5: make-adder 5
add10: make-adder 10

a: 30

add5 5
; returns: 10

add10 5
; returns: 15
```

## Contexts and Console

Above, we've seen a lot of what first class contexts can do and be used for. But context is also something we can just **navigate** in similar way as you navigate directories on your hard drive - _it's all just information anyway_ - with our Rye Console.


```clojure
; list current context
ls
; might be empty or not, depends if you were evaluating code above

; list parent context for words matching "math"
lsp\ "math"
; prints:
;  Context:
;   math: [ ....
;           .... ]

; change context to math
cc math

; list current context (now in math)
ls
; prints:
;  Context:
;   abs: [BFunction(1): ...]
;   acos: [BFunction(): ...]
;   ...


; Use the functions in the math context, calc is the math *dialect*
calc { abs ( 10 - 30 ) + 5 * 3 }

; change context back to parent (our initial context)
ccp

; makes and changes to new context todo
mkcc 'todo

; defines words in this context
todos: { "fix A" }
add: fn1 { .append! 'todos }

; list this todo subcontext
ls

; try using it
add "solve B"

display todos

; change back to parent
ccp

; call a function on it from our initial context
todo/add "improve C"

display todo/todos

; etc ...

```



