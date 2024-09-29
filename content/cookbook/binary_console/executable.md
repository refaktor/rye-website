---
title: "Rye binary*"
weight: 50
---

*work in progress*

# Rye --help

As first step, let's look at what we get if we run `rye --help`

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
  rye                                  # enters console/REPL
  rye -do "print 33 * 42"              # evaluates the do code
  rye -do 'name: "Jim"' console        # evaluates the do code and enters console
  rye cont                             # continues/loads last saved state and enters console
  rye -do 'print 10 + 10' cont         # continues/loads last saved state, evaluates do code and enters console
  rye filename.rye                     # evaluates filename.rye
  rye .                                # evaluates main.rye in current directory
  rye some/path/.                      # evaluates main.rye in some/path/
  rye -do 'print "Hello" path/.        # evaluates main.rye in path/ and then do code
  rye -console file.rye                # evaluates file.rye and enters console
  rye -do 'print 123' -console .       # evaluates main.rye in current dir. evaluates do code and enters console
  rye -silent                          # enters console in that doesn't show return values - silent mode
  rye -silent -console file.rye        # evaluates file.re and enters console in silent mode
  rye -lang eyr                        # enter console of stack based Eyr language
  rye -lang math                       # enter console of math dialect

 Thank you for trying out Rye ...

```

&nbsp;

# Basic behaviour

## Run Rye console

To run the console just call the binary with no argumetn

```
rye
```

## Run Rye script

Run rye script in file script.rye

```
rye script.rye
```

&nbsp;

# The -do flag

## Sum two numbers

To run the console just call the binary with no argumetn

```
rye -do '1234 + 2345 |print'
```

&nbsp;

## Load csv, get average of column height

To run the console just call the binary with no argumetn

```lisp
rye -do "load\csv %school.csv |autotype 1.0 |column? 'heigth |avg |print"
```

