---
title: "Rye binary*"
weight: 50
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
weight: 50
summary: "An growing list of simple GUI examples."
mygroup: true

---

*work-in-progress*

Rye binary tries to not be just a language runtime, but a useful tool. You can find the files used in these examples in `examples/cookbook/`.

# Rye --help

We'll run `rye --help` first and see what we get. Rye follows the default Go flags **convention** where flags can have one or two "-" to be valid.

```
╭────────────────────────────────────────────────────────────────────────────────────────────---
│ Rye language. Visit https://ryelang.org to learn more.                            
╰───────────────────────────────────────────────────────────────────────────────────────---

 Usage: rye [options] [filename or command]

 To enter Rye console provide no filename or command.

 Options (optional)
  -console
    	Enters console after a file is evaluated.
  -do string
    	Evaluates code after it loads a file or last save.
  -help
    	Displays this help message.
  -lang string
    	Select a dialect / language (rye, eyr, math) (default "rye")
  -silent
    	Console doesn't display return values

 Filename: (optional)
  [filename]   
       Executes a Rye file
  .            
       Executes a main.rye in current directory
  [some/path]/.
       Executes a main.rye on some path

 Commands: (optional)
  cont
     Continue console from the last save
  here
     Starts in Rye here mode (wip)
 Examples:
  rye                               # enters console/REPL
  rye -do "print 33 * 42"           # evaluates the do code
  rye -do 'name: "Jim"' console     # evaluates the do code and enters console
  rye cont                          # continues/loads last saved state and enters console
  rye -do 'print 10 + 10' cont      # continues/loads last saved state, evaluates do code and enters console
  rye filename.rye                  # evaluates filename.rye
  rye .                             # evaluates main.rye in current directory
  rye some/path/.                   # evaluates main.rye in some/path/
  rye -do 'print "Hello" path/.     # evaluates main.rye in path/ and then do code
  rye -console file.rye             # evaluates file.rye and enters console
  rye -do 'print 123' -console .    # evaluates main.rye in current dir. evaluates do code and enters console
  rye -sdo 'print 123'              # silent do command
  rye -silent                       # enters console in that doesn't show return values - silent mode
  rye -silent -console file.rye     # evaluates file.re and enters console in silent mode
  rye -lang eyr                     # enter console of stack based Eyr language
  rye -lang math                    # enter console of math dialect
  rye -ctx os                       # enter console and set os context as parent
  rye -ctx 'os pipes'               # enter console and set os and then pipes contexts as parent

 Thank you for trying out Rye. Rye is still work in progress, please reports any bugs you find.

```

&nbsp;

## Run Rye console (REPL)

To run the console (REPL) just call the binary with no arguments.

```
# run normal Rye console
rye

# run Rye console without displaying the return values
rye -silent
```

## Run Rye program

Run rye program in file hello.rye

```
rye hello.rye
# evaluates the hello.rye file

rye .
# evaluates the main.rye in the current directory


rye path/to/.
# evaluates main.rye on the path
```

## -do flag

Do flag evaluates the code you enter after it prints the result and exits. You can combine it with other flags for more versatile behaviour.

We can use `-do` flag and enter a simple Rye expression.

```bash
rye -do '1234 + 2345'
# prints 3579
```
&nbsp;

Here we load a csv file, get the number of rows, the headers and average of a column 'Soc score'.


```bash
rye -do "load\csv %friends.csv |length?"
# prints: 3

rye -do "load\csv %friends.csv |headers?"
# prints: L[ Id  Name  Age  Social Score ]

rye -do 'load\csv %friends.csv |autotype 1.0 |column? "Soc score" |avg'
# prints: 75.666667
```
&nbsp;

### -do and -silent flags

Evaluate the `-do` code withot printing the result. 

`-do` flag normally prints a result, so without the -silent 
flag we would print last 17 twice, because that is also a returned value.

```bash
# print out the number of characters for each line in a file

rye -do 'read\lines %friends.csv |for { .length? print }' -silent
# prints:
# 21
# 16
# 13
# 17
```
&nbsp;

### -do and here command

Here command evaluates the local .rye-here file and then the -do code. Here we store our helper function
into rye-here and then use it three t

```bash
cat > .rye-here
load-csv: does { load\csv %friends.csv }

rye -do "load-csv |columns? { 'Id 'Age } |save\csv %ages.csv" here
# creates file ages.csv

rye -do 'load-csv |where-lesser "Soc score" 50 |save\csv bad.csv' here
# creates file bad.csv

rye -do "load\csv %bad.csv |to-json |post* https://distop.ia 'json"
# makes HTTP POST with json to distop.ia
```

&nbsp;

### -do and -console flags

Do the code and then enter Rye console.

```bash
rye -do "load\csv %friends.csv :friends" -console
# evaluates the code and enters console
x> spr .where-greater "Age" 50 |display
x> ...
```

&nbsp;

### -do flag and a filename

In this case Rye evaluates the filename and then does the `-do` code. This lets you evaluate a filename, but enter console after it, so you
can interact with it from Rye console

```bash
cat > friends.rye
spr: load\csv %friends.csv
save: fn { s } { .save\csv %friends.csv }


rye -do 'spr .add-column! { 191 "Bob D" 72 133 } |save' friends.rye

rye -do 'spr .display' friends.rye
```

&nbsp;

## -console flag

This flag forces that you get into console. So you can avaluate a rye file and after it get to the console. 

```bash
# we use the friends.rye file we create above

rye -console friends.rye
# We evaluated a file friends.rye, then we eneter the console, so we can work with data interactv
```

## Save current state and continue

Sometimes you work in Rye console and you have to end. You can save the state you are in with function `save\current`.
This saves the "data", variables, but also functions you defined. To enter back into console use th `cont` (continue) 
command.

```bash
# enter Rye console
rye
x> name: "Jim Beam"
x> say-hi: fn { x } { print "Hi " + x }
x> save\current
# saved do file console_<date>_<time>.

# continue Rye console from last saved state
rye continue
x> lc                    ; list context
Context:
 name: [String: Jim]
 say-hi: [Function(1)]
x> say-hi name
Hi Jim Beam
```

## -lang flag

Rye runtime has multiple evaluators / dialects. While ones are highly specialised (like validation or sql) there are more general ones, that can
be used on their own. 


```bash
# start rye console with Eyr (a stack based dialect)
rye --lang eyr
x> 1010
[Stack: ^[Integer: 1010] ]
x> 101
[Stack: ^[Integer: 1010] [Integer: 101] ]
x> +
[Stack: ^[Integer: 1111] ]
x> 2 3 + *
[Stack: ^[Integer: 5555] ]
x> .
[Stack: ]
x> { x y } { y x } fn :swap
[Stack: ]
x> { "Hi " swap concat print . } does :say-hi
[Stack: ]
x> "Anne" say-hi
; prints: Hi Anne
```

## -ctx flag

Rye runtima also holds multiple contexts and you can start a Rye console as a specific context as a parent. You can also define multiple context and Rye console will
start with that chain of contexts as parents.


```bash
# start rye console with os context as parent
# this gives you functions like ls, cd, mkdir
# use lcp to explore the functions
rye --ctx 'os pipes'
x> mkdir %test |cd
[Uri: file://test]
x> cwd
[Uri: file:///home/ryefaktor/rye/examples/cookbook/test]
x> echo "World" |into-file %hello.txt
[Integer: 5]
x> ls
[Block: ^[Uri: file://hello.txt] ]
```
