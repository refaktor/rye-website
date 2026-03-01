---
title: "Email/SMTP/IMAP"
weight: 250
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Sending, reading, parsing and constructing email."
mygroup: true

---

*work-in-progress*

Rye has builtins for sending email via SMTP, reading email via IMAP, and parsing email files. This page covers the main operations.

## Sending email (SMTP)

Use the gomail-based builtins to compose and send emails via SMTP.

### Creating and configuring a message

```clojure
; Create a new email message
email-message

; Set headers
|Set-header "Subject" "Hello from Clojure"
|Set-header 'To "recipient@example.com"
|Set-header 'From "sender@example.com"

; Or use address header with display name
|Set-address-header "From" "sender@example.com" "John Doe"
|Set-address-header "To" "recipient@example.com" "Jane Smith"

; Set the body - plain text
|Set-body "text/plain" "Hello, this is the email body."

; Or HTML body
|Set-body "text/html" "<h1>Hello</h1><p>This is HTML content.</p>"
```

### Multipart emails (text + HTML)

```clojure
email-message
|Set-header "Subject" "Multipart email"
|Set-address-header "From" "sender@example.com" "Sender"
|Set-address-header "To" "recipient@example.com" "Recipient"

; Set plain text as primary body
|Set-body "text/plain" "Plain text version for simple clients."

; Add HTML as alternative
|Add-alternative "text/html" "<p>HTML version with <b>formatting</b>.</p>"
```

### Adding attachments

```clojure
email-message :msg
|Set-header "Subject" "Email with attachment"
|Set-header 'To "recipient@example.com"
|Set-body "text/plain" "Please see the attached file."

; Attach a file
|Attach %file://document.pdf
|Attach %file://image.jpg
```

### Sending the email

```clojure
; Create SMTP dialer with server details
dialer: new-email-dialer "smtp.gmail.com" 587 "user@gmail.com" "app-password"

; Send the message
dialer .Dial-and-send msg
```

## Parsing email files

Parse email files (EML format) to extract content.

### Basic parsing

```clojure
; Parse email from file
Reader %email.eml |Parse-email :eml

; Extract fields
print Subject? eml
print Text-body? eml
print Html-body? eml
print Message-id? eml
```

### Example script for parsing

```clojure
; main.clojure - parse email file from command line arg
file: first args

Reader to-file file |Parse-email :eml

print "Subject: " ++ Subject? eml
print "---"
print Text-body? eml
```

Run with: `clojure main.clojure myemail.eml`

## Reading email (IMAP)

Connect to an IMAP server to read emails from mailboxes.

### Connecting

```clojure
; Connect with username/password
client: imap-client "user@gmail.com" "app-password" "imap.gmail.com" 993

; Or connect with OAuth2
client: imap-client\oauth2 "user@gmail.com" "access-token" "imap.gmail.com" 993
```

### Listing folders

```clojure
client: imap-client "user@gmail.com" "password" "imap.gmail.com" 993

; Get list of available folders
folders: client .Get-folders
folders |for { .print }
; prints: INBOX, Sent, Drafts, etc.
```

### Selecting a folder

```clojure
; Select the inbox
client .Select-folder "INBOX"

; Or other folders
client .Select-folder "Sent"
client .Select-folder "Drafts"
```

### Searching for emails

Search using IMAP search criteria and get UIDs of matching emails:

```clojure
; Search for unread emails
uids: client .Search-emails "UNSEEN"

; Search by sender
uids: client .Search-emails "FROM \"sender@example.com\""

; Search by subject
uids: client .Search-emails "SUBJECT \"Important\""

; Search by date (IMAP date format: DD-Mon-YYYY)
uids: client .Search-emails "SINCE \"01-Jan-2024\""
```

### Getting email overviews (fast)

Get headers only - fast for listing many emails:

```clojure
uids: client .Search-emails "UNSEEN"
overviews: client .Get-overviews uids

; Each overview is a dict with: uid, subject, from, to, date, size, flags
overviews |for { ::email
    print email -> "subject"
    print email -> "from"
    print "---"
}
```

### Getting full emails

Get complete email content including bodies and attachments:

```clojure
uids: client .Search-emails "UNSEEN"
emails: client .Get-emails uids

; Each email dict has: uid, subject, from, to, cc, bcc, date, 
; received, message-id, size, text, html, flags, attachments
emails |for { ::email
    print "Subject: " ++ email -> "subject"
    print "From: " ++ email -> "from"
    print "Text: " ++ email -> "text"
    print "---"
}
```

### Managing emails

```clojure
; Mark email as read
client .Mark-seen 12345

; Move email to another folder
client .Move-email 12345 "Archive"

; Delete email (marks for deletion)
client .Delete-email 12345

; Permanently remove deleted emails
client .Expunge
```

### Closing the connection

```clojure
client .Close
```

### Complete IMAP example

```clojure
; Connect and process unread emails
client: imap-client "user@gmail.com" "app-password" "imap.gmail.com" 993

client .Select-folder "INBOX"
uids: client .Search-emails "UNSEEN"

either uids .length? > 0 {
    print "Found " ++ to-string length? uids ++ " unread emails"
    
    emails: client .Get-emails uids
    emails |for { ::email
        print "Subject: " ++ email -> "subject"
        print "From: " ++ email -> "from"
        
        ; Mark as read after processing
        client .Mark-seen email -> "uid"
    }
} {
    print "No unread emails"
}

client .Close
```
