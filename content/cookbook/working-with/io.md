---
title: "IO (Input Output)*"
weight: 70
date: 2024-11-21T19:30:08+10:00
draft: false
group: true
summary: "If a program is running, and there is no IO, is it even heating the CPU?"
mygroup: true

---

*work-in-progress*

## Working with file paths

### Reading a file

```lisp
$ cat > test.txt
can we all be happy!
also the sad people! :P

$ rye

read %test.txt
; returns string with two lines

read\lines %test.txt
; returns block with two strings
; one for each line
```
&nbsp;

### Writing to a file

```lisp
"Save me to a disc"
|write %save.txt

"Me too"
|write\append %save.txt   ; TODO
```


## Copying a file

Rye can copy a file directly from reader to writer without loading it full into memory.

```lisp
$ cat > source.txt
hello copy!

$ rye

open %source.txt |copy create %destination.txt
```

## Appending to file

Append next number to a file each second, forever.

```lisp
open\append %seconds.txt :wr

forever { .to-string + "\n" |write\string* wr , sleep 1000 }

[Ctrl-z]

$ tail -f seconds.txt
```
&nbsp;

_TODO: #bug Ctrl-z or Ctrl-c don't work when in forever loop._

## Tailing a file

The `tail -f` opens a file and waits for appends to a file and prints them. We can also to this in Rye, retrieving additions to a file as they come.

```lisp
open %seconds.txt :f
.seek\end
.reader :r
forever { read\string r "\n" |print }
```

## HTTP Download

Downloading a bigger file without loading it into memmory. Open with htts URI opens a reader, and copy copies (streams) it over to a writer, which we can create with create and a file path.

```lisp
open https://fu.gov.si/fileadmin/prenosi/DURS_zavezanci_PO_csv.zip
|copy open %download.zip
```

## Downloading from FTP

FTP is more complex (stateful?) protocol, but we can use it similarly as HTTP above to download big files with directly streaming them to disc for example.

```lisp
; load the ftp username and password from a file
info: context { do load %.ftpinfo }

open ftp://prenos.example.com:21
|login info/user info/pwd
|retrieve "rtr/MES/BIG_FILE.zip"
|copy create %local3.zip
```

