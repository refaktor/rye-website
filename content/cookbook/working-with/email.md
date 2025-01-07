---
title: "Email*"
weight: 250
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Sending, reading, parsing and constructing email."
mygroup: true

---

*work-in-progress*

Rye currently had multple ways to send email (smtp, amazon ses, postmark), to receive email via smtp. First we will document parsing various forms of email, bacause I concretely need this.

## Parsing email

### Plain text email

```lisp

$ wget https://ryelang.org/cookbook/working-with/plain-text.eml

$ rye -history main.rye -console -args plain-text.eml

file: rye .arg?

reader to-file file |parse-email :eml
eml .subject? |print
eml .text-body? |print
eml .html-body? |print
```

we got main.rye script from our history. Now we can use it for other files too

### Multipart email with text part

```lisp
$ wget https://ryelang.org/cookbook/working-with/multipart-1.eml

$ rye . multipart-1.eml |html->markdow  
```

### Multipart email with just HTML


```lisp




```

### Attachments

```lisp




```

### Odd cases

```lisp




```


