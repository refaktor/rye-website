---
title: "Testing"
date: 2024-10-24T19:30:08+10:00
draft: false
group: true
weight: 250
summary: "Testing Rye runtime and base library"
mygroup: true
---

# Running tests

You can enter the `tests/` folder and since there is a `main.rye` you can run it with a dot and you will get the following:

```
(mycomp) -> cd tests/
(mycomp) -> rye .
# Rye's simple testing tool
	
 use test or doc command

Examples: 
 rye . test           # runs all tests
 rye . doc            # generates the docs out of tests
 rye . test ls        # lists the test sections available
 rye . test base      # runs a specific test section
```
&nbsp;

Example of use:

![usage and running a test](../testing_passed.png)

## Test sections

The test sections are organized thematically:

- **base** - Core builtins (boolean, numbers, strings, time, collections, vectors, types, contexts, conditionals, iteration, functions, combinators, printing, error handling)
- **table** - Table/spreadsheet operations
- **formats** - Regexp, JSON, BSON, SXML, HTML, Markdown, conversion
- **io** - File I/O, command execution, SQLite, PostgreSQL, MySQL
- **crypto** - Cryptographic functions and bcrypt
- **dialects** - Math, validation, match, Eyr dialect, complex numbers
- **protocols** - HTTP, email, mail, IMAP, SMTPD, MQTT
- **system** - OS operations, Git, SSH, goroutines
- **web** - Web-related builtins
- **pipes** - Unix-style pipe operations

Run a specific section:

```
(mycomp) -> rye . test base

# BASE #

Boolean
 Functions that work with true and false values.
 true ✓ ✓ 
 false ✓ ✓ 
 not ✓ ✓ ✓ ✓ 
 ...
```
&nbsp;

# Regenerating info files

The `./regen` script in the tests/ folder regenerates all .info.rye files from the Go source code comments. 
The script uses the `cmd/rbit/rbit` tool to parse Go code and extract test definitions.

```bash
cd tests/
./regen
```

The regen script groups multiple builtin files into themed info files:

```bash
# base.info.rye combines many core modules:
../cmd/rbit/rbit ../evaldo/builtins_base_boolean.go > base.info.rye
../cmd/rbit/rbit ../evaldo/builtins_base_numbers.go >> base.info.rye
../cmd/rbit/rbit ../evaldo/builtins_base_strings.go >> base.info.rye
# ... (more modules appended)

# formats.info.rye combines format-related modules:
../cmd/rbit/rbit ../evaldo/builtins_regexp.go > formats.info.rye
../cmd/rbit/rbit ../evaldo/builtins_json.go >> formats.info.rye
../cmd/rbit/rbit ../evaldo/builtins_bson.go >> formats.info.rye
# ... etc
```

&nbsp;

# Generating HTML Reference

The same `tests/main.rye` script can generate HTML documentation from the .info.rye files:

```bash
cd tests/
rye . doc
# docs generated
```

This creates HTML files (base.html, table.html, formats.html, etc.) using templates from `tests/tpl/`. 
The generated docs are available online at https://ryelang.org/info

&nbsp;

# Test framework internals

The `main.rye` defines two contexts:

- **test-framework** - Runs tests and reports results (equal, error, stdout assertions)
- **docs-framework** - Generates HTML documentation from the same test definitions

This means the same .info.rye files serve both as executable tests and documentation source!

Example .info.rye structure:

```rye
section "Boolean " "Functions that work with true and false values." {
    group "true" 
    "Returns a boolean true value."
    {
        arg `none`
        returns `boolean true value`
    }
    {
        equal { true } true
        equal { true |type? } 'boolean
    }
    { `` }
}
```

Each `group` block contains:
1. Function name and docstring
2. Arguments and return value specification  
3. Test assertions (equal, error, stdout)
4. Optional example code

# Tests and "one source of truth"

Tests are provided as comments above builtin function definitions in Go files. This keeps tests close to code.
See the [One source page](../one-source/) for more information about how tests, documentation, and examples 
are unified in one place.
