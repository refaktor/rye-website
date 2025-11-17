---
title: "Rye Executable"
weight: 50
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Using the Rye binary as a command-line tool."
mygroup: true

---

The Rye executable is more than just a language runtime - it's a versatile command-line tool for evaluation, scripting, and interactive exploration.

## Basic usage

```bash
rye                    # Interactive console (REPL)
rye script.rye         # Execute a Rye script
rye .                  # Execute main.rye in current directory
rye -do "1 + 2"        # Evaluate expression and exit
```

## Command-line flags

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

Evaluate the `-do` code without printing the result. 

The `-do` flag normally prints a result, so without the `-silent` 
flag we would print the last value twice.

```bash
# Print the number of characters for each line in a file
rye -do 'Read\lines %friends.csv |for { .length? .print }' -silent
# prints:
# 21
# 16
# 13
# 17
```

### -do and here command

The `here` command evaluates the local `.rye-here` file and then the `-do` code. Store helper functions in `.rye-here` and reuse them:

```bash
cat > .rye-here
load-csv: does { load\csv %friends.csv }

rye -do "load-csv |columns? { 'Id 'Age } |save\csv %ages.csv" here
# creates file ages.csv

rye -do 'load-csv |where-lesser "Soc score" 50 |save\csv %bad.csv' here
# creates file bad.csv
```

### -do and -console flags

Execute code and then enter the Rye console:

```bash
rye -do "friends: load\csv %friends.csv" -console
# evaluates the code and enters console
x> friends .where-greater "Age" 50 |display
x> ...
```

### -do flag with filenames

Rye evaluates the file first, then executes the `-do` code. This allows you to run a script and then interact with its environment:

```bash
cat > friends.rye
spr: load\csv %friends.csv
save: fn { s } { .save\csv %friends.csv }

rye -do 'spr .add-column! { 191 "Bob D" 72 133 } |save' friends.rye
rye -do 'spr .display' friends.rye
```

## -console flag

Forces entry into the console after evaluating a file. Useful for debugging and interactive exploration:

```bash
rye -console friends.rye
# Evaluates friends.rye, then enters console for interactive work
```

## State persistence

Save and restore console sessions with all variables and functions:

```bash
# Enter Rye console
rye
x> name: "Jim Beam"
x> say-hi: fn { x } { print "Hi " + x }
x> save\current
# Saves to console_<date>_<time>.rye

# Continue from last saved state
rye cont
x> lc                    ; list context
Context:
 name: [String: Jim]
 say-hi: [Function(1)]
x> say-hi name
Hi Jim Beam
```

## Language dialects

Rye supports multiple evaluators and dialects. Use `-lang` to switch between them:

```bash
# Stack-based Eyr dialect
rye -lang eyr
x> 1010 101 +
[Stack: ^[Integer: 1111] ]
x> 2 3 + *
[Stack: ^[Integer: 5555] ]

# Math-focused dialect
rye -lang math
```

## Context selection

Start the console with specific contexts loaded, giving you access to specialized functions:

```bash
# Start with OS context (ls, cd, mkdir, etc.)
rye -ctx os
x> ls |first

# Chain multiple contexts
rye -ctx 'os pipes'
x> mkdir %test |cd
x> echo "Hello" |into-file %hello.txt
x> ls
```

Use `lcp` in the console to explore available functions in the loaded contexts.
