---
title: "IO (Input Output)"
weight: 70
date: 2024-11-21T19:30:08+10:00
draft: false
group: true
summary: "If a program is running, and there is no IO, is it even heating the CPU?"
mygroup: true

---

*work-in-progress*

## File paths and operations

Rye file names and file paths are prefixed with `%`, which is a shorthand for **file-uri**. `%somefile.txt` is a shorthand for `file://somefile.txt`. So we are using methods on `file-uri` Kind.

Use `lg 'file-uri` to see all related methods.

### Analyzing Path

```clojure
; Get file extension (including the dot)
File-ext? %data/file.txt        ; returns ".txt"
File-ext? %data/file.temp.png   ; returns ".png"

; Get filename with extension
Filename? %path/to/document.pdf ; returns "document.pdf"

; Get filename without extension (stem)
Stem? %path/to/document.pdf     ; returns "document"

; Get directory path
Dir? %path/to/document.pdf      ; returns "path/to"

; Split path into components
Split %path/to/document.pdf     ; returns { "path" "to" "document.pdf" }

; Check if path is absolute
Is-absolute %/home/user/file.txt  ; returns true
Is-absolute %relative/path.txt    ; returns false

; Check if file exists
Does-exist %data/file.txt         ; returns true or false
```

### Reading files

```clojure
; Read entire file as string
Read %test.txt
; returns string with full content

; Read file as block of lines
Read\lines %test.txt
; returns block with one string per line

; Read file as bytes
Read\bytes %binary.dat
; returns native bytes object
```

### Writing files

```clojure
; Write string to file (creates or overwrites)
Write %output.txt "Hello, World!\n"
; returns the content that was written
```

### File natives

For more control, you can work with opened file natives directly. These are of kind file. Use `lg 'file` to see all related methods.

```clojure
; Create a new file and write to it
Create %output.txt
|Write "Hello, World!\n"
|Write "Second line\n"
|Close

; Open existing file for reading
Open %data.txt :file
|Read :content
Close file
print content

; Get file information
file: Open %data.txt
info: Stat file
print "File size:" ++ to-string Size? info
file .Close
```

## Reader/Writer pattern

Rye uses a reader/writer pattern for streaming data. This allows processing large files without loading everything into memory.

### Creating readers

```clojure
; Reader from file-uri
reader: Reader %data.txt

; Reader from file object
Open %data.txt |Reader

; Reader from string (for testing/processing)
reader "some string content"
```

```bash
# Standard input, example from bash
$ echo "Hello world" | rye -do 'stdin |Read\string'
```

### Creating writers

```clojure
; Writer from file object
; Here we use defer variation that lets us
; set deferred close of native inline
Create %output.txt |defer\ 'Close
|Writer |defer\ 'Close
|Write "some\n" 
|Write "data\n"
```

```bash
$ rye -silent -do 'stdout |Write "hello\nawk\n"' | awk '{ print $1 }'
# hello
# awk
```

### Reading content

```clojure
; Read all content as string
Reader %data.txt :reader |Read\string
reader .Close
print content
```

### Copying between reader and writer

```clojure
; Copy without loading into memory
Open %source.txt |Copy Create %destination.txt

; Or using reader explicitly
reader: Reader %source.txt
writer: Writer Create %dest.txt
reader .Copy writer
```

## Appending to files

```clojure
; Open file for appending
Open\append %log.txt |defer\ 'Close
|.Write "New log entry\n"
```

## File tailing (monitoring)

The `tail-file` builtin monitors files for new content - useful for log file monitoring:

```clojure
; Tail a file, following new contenta
tailer: tail-file %app.log true true
; Arguments: path, follow (true), reopen on rotation (true)

; Read lines as they are added
forever {
    line: tailer .Read-line
    print "New line: " ++ line
}

; Close when done
tailer .Close
```

## Console input

```clojure
; Prompt user for input
name: input "Enter your name: "
print "Hello, " ++ name
```

<!-- ## Bytes operations

```clojure
; Read file as bytes
bytes1: Read\bytes %file1.bin

; Append two byte arrays
combined: append\bytes bytes1 bytes2

; Write bytes to file
write\bytes combined "output.bin"
```
-->
