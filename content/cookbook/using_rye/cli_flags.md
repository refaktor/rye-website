---
title: "More on flags"
weight: 180
date: 2025-03-29T19:30:08+10:00
draft: false
group: true
summary: "Rye binary commandline flags and their meaning"
mygroup: true

---

# Rye Command Line Flags

Rye is a versatile programming language with a rich set of command line flags that can be combined in various ways to enhance your development experience. This guide explores the many flags available in Rye and demonstrates how they can be used together to solve different problems.

## Basic Command Line Flags

| Flag | Description |
|------|-------------|
| `-do <code>` | Evaluates Rye code after loading a file or last save |
| `-sdo <code>` | Same as `-do` but in silent mode (doesn't display return values) |
| `-lang <dialect>` | Select a dialect/language (rye, eyr, math) |
| `-ctx <context>` | Enter a specific context or context chain |
| `-silent` | Console doesn't display return values |
| `-stin <mode>` | Inject first value from stdin (modes: no, all, a) |
| `-console` | Enters console after a file is evaluated |
| `-dual` | Starts REPL in dual-mode with two parallel panels |
| `-template` | Process file as a template, evaluating Rye code in {{ }} blocks |
| `-help` | Displays help message |

## Security-Related Flags (Linux only)

| Flag | Description |
|------|-------------|
| `-seccomp-profile <profile>` | Seccomp profile to use: strict, readonly |
| `-seccomp-action <action>` | Action on restricted syscalls: errno, kill, trap, log |

## Command Patterns

Rye supports several command patterns:

- `rye` - Enters the Rye console (REPL)
- `rye <filename>` - Executes a Rye file
- `rye .` - Executes main.rye in the current directory
- `rye <path>/.` - Executes main.rye in the specified path
- `rye continue` or `rye cont` - Continues console from the last save
- `rye here` - Starts in Rye here mode (loads .rye-here file)

## Powerful Flag Combinations

Let's explore how these flags can be combined to create powerful workflows:

### 1. Development and Debugging

#### Interactive Development with Files

```bash
# Load a file and enter console to interact with its contents
rye -console myprogram.rye

# Load a file, execute additional code, then enter console
rye -do 'debug-mode: true' -console myprogram.rye

# Continue from last saved state with debugging enabled
rye -do 'debug-mode: true' continue
```

#### Context-Specific Development

```bash
# Start console with OS context for file operations
rye -ctx os

# Start console with multiple contexts chained
rye -ctx 'os pipes'

# Load a file in a specific context and enter console
rye -ctx 'os' -console file-processor.rye
```

### 2. Data Processing Workflows

#### CSV Processing

```bash
# Load CSV, process it, and save results
rye -do 'load\csv %data.csv |where-greater "Age" 50 |save\csv %filtered.csv' -silent

# Load CSV, analyze it, and enter console for further exploration
rye -do 'data: load\csv %data.csv |autotype 1.0' -console

```

<!--
; TODO
; # Process multiple files in sequence
; rye -do 'glob "*.csv" |for { .load\csv |process |save\csv (+ "processed_" .) }' -silent
-->

#### Combining with Here Mode

```bash
# Create a .rye-here file with common functions
# Then use those functions in one-liners
rye -do 'load-data |transform |save-results' here
```

### 3. Security Scenarios

#### Readonly Operations

```bash
# Run a script that should only read files, not write them
rye -seccomp-profile=readonly data-analyzer.rye

# TODO:
# Combine with strict action for maximum security
# rye -seccomp-profile=readonly -seccomp-action=kill data-analyzer.rye

# Allow reading but prevent writing in interactive mode
rye -seccomp-profile=readonly -console
```

#### Strict Security Profile

```bash
# Run with strict security profile (prevents dangerous syscalls)
rye -seccomp-profile=strict my-script.rye

# TODO
# Debugging security issues with logging
# rye -seccomp-profile=strict -seccomp-action=log my-script.rye
```

### 4. Template Processing

```bash
# Process a template file, evaluating Rye code in {{ }} blocks
rye -template my-template.txt

# Process template with specific context
rye -ctx 'os' -template my-template.txt

# Process template and save result
rye -do 'result: process-template %my-template.txt; write-file %output.txt result' -silent
```

### 5. Language Dialects

```bash
# Use the stack-based Eyr dialect
rye -lang eyr

# Run a math-focused script with the math dialect
rye -lang math calculations.rye

# Combine dialect with context
rye -lang eyr -ctx 'os' -console
```

<!--

### 6. Advanced Console Usage

```bash
# Start in silent mode for cleaner output
rye -silent

# Dual-panel REPL for comparing operations
rye -dual

# Load from stdin and process in console
cat data.txt | rye -stin all -console
```

### 7. Automation and Scripting

```bash
# One-liner for quick data transformation
rye -do 'read-file %input.txt |transform |write-file %output.txt' -silent

# Chain multiple operations
rye -do 'for glob "*.json" { |read-file |parse\json |process |to-json |write-file (+ "processed_" .) }' -silent

# Create a report from data
rye -do 'data: load\csv %data.csv; report: generate-report data; write-file %report.md report' -silent
```

## Real-World Examples

### Example 1: Data Analysis Pipeline

```bash
# Create a .rye-here file with common functions
cat > .rye-here << EOF
load-data: does { load\csv %sales.csv |autotype 1.0 }
summarize: does { x } {
  .group-by "Region" |map { 
    region: .key
    sales: .values |column? "Sales" |sum
    count: .values |length?
    object { region sales count }
  }
}
EOF

# Run a quick analysis
rye -do 'load-data |summarize |to-json |write-file %summary.json' here -silent

# Or enter console for interactive analysis
rye -do 'data: load-data' -console here
```

### Example 2: Secure Log Analyzer

```bash
# Create a log analyzer that can only read files
cat > log-analyzer.rye << EOF
cc os

logs: glob "*.log" |map { read-file }
patterns: [ "ERROR" "WARNING" "CRITICAL" ]

for logs { log } {
  for patterns { pattern } {
    matches: find-all log pattern
    if (length? matches) > 0 {
      print "Found " + (length? matches) + " " + pattern + " entries in " + log
    }
  }
}
EOF

# Run with readonly security profile
rye -seccomp-profile=readonly log-analyzer.rye
```

### Example 3: Template-Based Report Generator

```bash
# Create a template
cat > report-template.txt << EOF
# Sales Report: {{ date\now |format "%Y-%m-%d" }}

## Summary

Total Sales: ${{ load\csv %sales.csv |column? "Amount" |sum |format "%.2f" }}
Average Sale: ${{ load\csv %sales.csv |column? "Amount" |avg |format "%.2f" }}
Number of Sales: {{ load\csv %sales.csv |length? }}

## Top Performers

{{ load\csv %sales.csv |sort-by "Amount" true |take 3 |table }}
EOF

# Generate the report
rye -template report-template.txt > sales-report.md
```

### Example 4: Interactive Data Exploration with Context

```bash
# Start with OS and pipes contexts, load data, and explore
rye -ctx 'os pipes' -do 'data: load\csv %data.csv |autotype 1.0' -console

# In the console:
# x> data |column? "Revenue" |stats
# x> data |where-greater "Revenue" 1000 |save\csv %high_revenue.csv
# x> exit
```

## Tips and Tricks

1. **Combine `-do` with `here`** to create reusable workflows with common functions defined in `.rye-here`

2. **Use `-sdo` instead of `-do -silent`** for more concise silent operations

3. **Save console state with `save\current`** and resume with `rye continue`

4. **Chain contexts with spaces** in the `-ctx` flag: `-ctx 'os pipes http'`

5. **Use seccomp profiles** for added security when running untrusted scripts

6. **Process templates** for dynamic document generation

7. **Explore different dialects** for specialized tasks (eyr for stack-based programming, math for mathematical operations)

8. **Use dual-mode REPL** for comparing operations side by side

## Conclusion

Rye's command line flags offer tremendous flexibility and power. By combining these flags in creative ways, you can build efficient workflows for development, data processing, security, and automation. Experiment with different combinations to discover the approach that works best for your specific needs.

-->
