---
title: "HTTP Servers"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 50
summary: "A growing list of server-side examples."
mygroup: true
---

<style>
  pre > code {
      __width: 440px;
      border-radius: 5px;
      __padding: 16px;
      __background-color: #3e3e3e;
      font-size: 13px;
  }
</style>

*work in progress*

## Hello world over HTTP

This is the simplest HTTP server example in Rye. It demonstrates the basic building blocks: creating a server, handling a route, and serving responses.

What's new: `http-server` `handle` `write` `serve`

```lisp
rye .needs { http }

http-server ":8081"
|handle "/hello" fn { w r } { .write "Hello world" }
|serve

; or for static text responses
; |handle "/hello" "Hello world"
```

The `http-server` creates a new server instance listening on port 8081. The `handle` method defines a route handler that takes a response writer `w` and request `r` as arguments. The `serve` method starts the server and begins listening for requests.

## Serve folder over HTTP

Static file serving is essential for web applications that need to serve HTML, CSS, JavaScript, images, and other assets. This example shows how to serve an entire directory structure over HTTP.

Keywords: `http` `static` `folder`

```lisp
rye .needs { http }

http-server ":8085"
|handle "/" new-static-handler %public_html
|serve
```

The `new-static-handler` creates a file server handler that serves files from the specified directory. Any request to the server will look for corresponding files in the `public_html` folder.

## Serve load averages over HTTP

This example demonstrates how to create a simple API endpoint that returns system information in JSON format. It's useful for monitoring applications or health checks.

What's new: `http` `os` `load-avg?` `to-json` 

```lisp
rye .needs { http os }

http-server ":8086"
|handle "/" fn { w r } { do\par os { load-avg? |to-json |write* w } }
|serve
```

The `do\par os` switches to the operating system context to access system functions. `load-avg?` retrieves the system load averages, which are then converted to JSON format and written to the HTTP response.

## Websocket server

WebSockets enable real-time bidirectional communication between client and server. This example shows both a simple echo server and a more complex interactive server that responds differently based on user input.

What's new: `handle-ws` `read` `write`

```lisp
rye .needs { http }

http-server ":9090"
|handle-ws "/echo" fn { s ctx } { forever { read s |write* s } }
|handle-ws "/captcha" fn { s ctx } { 
	write s "Hasta la vista," 
	forever {
		read s |switch {
			"baby" { "*Welcome back, John Connor*" }
			_      { "Beep Boop" }
		} |write* s } }
|serve
```

The `handle-ws` method creates WebSocket handlers. The `read` function waits for messages from the client, while `write` sends messages back. The `forever` loop keeps the connection alive for continuous communication.

## Controlled file download

This example demonstrates secure file downloads by requiring authentication via a hashed key. It shows how to set custom HTTP headers, handle errors with appropriate status codes, and stream file contents to the client.

What's new: `write-header` `set-header` `copy`

Accept a key, hash it, if the file with that name exists return it.

```lisp
rye |needs { http bcrypt }

handle-file-get: fn { w r } {

	query? r "key" |^fix {
		write-header w 402
		write w "Err: Missing Key"
	}
	|bcrypt-hash :name

	open %storage/ + name
	|^fix {
		write-header w 403
		write w "Err: Wrong code"
	}
	:file .stat :info

	with w {
		.write-header 200 ,
		.set-header 'Content-Type "application/octet-stream" ,
		.set-header 'Content-Length probe to-string info .size? ,
		.set-header 'Content-Disposition "attachment; filename=mycert.p12"
	}

	file .copy w	
}

http-server ":8080"
|handle "/get" ?handle-file-get
|serve
```

The `query?` function extracts URL parameters, `write-header` sets HTTP status codes, and `set-header` adds custom headers. The `^fix` operator provides error handling when operations fail.



## Handle file uploads

File upload handling is crucial for many web applications. This example shows how to process multipart form data, extract uploaded files, and save them to disk with auto-generated filenames based on timestamps.

What's new: `parse-multipart-form!` `form-file?`

Serve static files on /static/ route and offer file upload on /upload

```lisp
rye .needs { http }

; generate path in uploads dir using curr. time,
; file extension. example: uploads/2023-03-06013601.png

gen-path: fn1 {
	.filename? .file-ext? :ext
	now .to-string .replace-all* regexp "[ :]" ""
	|concat* "uploads/" |+ ext |to-file 
}

; handle upload

upload: fn { w r } {
	parse-multipart-form! r
	form-file? r "img" |set [ file headers ]
	new-file: create gen-path headers 
	copy file new-file |^check { write w "Error" }
	write w "OK"
}

http-server ":8080"
|handle "/" new-static-handler %www
|handle "/upload" ?upload
|serve
```

The `parse-multipart-form!` function processes the multipart form data, while `form-file?` extracts the uploaded file and its metadata. Files are saved with unique timestamped names to avoid conflicts.

## Full text search API from JSON file

Keywords: `http` `API` `bleve` `search` `CSV`


```lisp
rye .needs { http bleve }

private\ "opens or creates a bleve index" {
        %companies.bleve :db |open-bleve
        |fix {
                new-bleve-document-mapping :fqm
                new-bleve-text-field-mapping :tfm

                new-bleve-index-mapping :im
                |add-document-mapping "companies" fqm
                |add-field-mapping-at "name" tfm
                |add-field-mapping-at "sname" tfm

                print "Index created, use: `bleve bulk companies.bleve data.json` to index in bulk"
                im .new-bleve db
        }
} :idx

find: fn1 { |probe |new-match-query |probe |new-search-request .search idx }

; enter-console "you can try searching in console"

handle-search: closure { w r } {
        find query? r "q" |probe |to-list idx |to-json |write* w
}

http-server ":9977" |handle "/" ?handle-search |serve
```

## Simple query parameter handling

This example shows how to extract and process URL query parameters, demonstrating basic request parsing and URL manipulation.

What's new: `query?` `url?` `path?`

```lisp
rye .needs { http }

handle-greet: fn { w r } {
	name: query? r "name" |fix { "Anonymous" }
	url? r |path? :current-path
	
	response: "Hello " + name + "! You visited: " + current-path
	write w response
}

http-server ":8083"
|handle "/greet" ?handle-greet
|serve

; Try: http://localhost:8083/greet?name=Alice
```

The `query?` function extracts named parameters from the URL query string. The `url?` and `path?` functions provide access to the request URL and its components.

## Cookie and session handling

This example demonstrates how to set and retrieve cookies, and manage user sessions with encrypted cookie storage.

What's new: `new-cookie-store` `get` `set` `save` `cookie-val?`

```lisp
rye .needs { http }

store: new-cookie-store "my-secret-key-change-in-production"

handle-login: fn { w r } {
	username: form? r "username" |fix { "guest" }
	session: get store r "user-session"
	set session "username" username
	save session r w
	write w "Logged in as: " + username
}

handle-profile: fn { w r } {
	session: get store r "user-session" 
	|^fix {
		write w "Please log in first"
	}
	username: get session "username" |^check { "missing session data" }
	write w "Welcome back, " + username + "!"
}

http-server ":8084"
|handle "/login" ?handle-login
|handle "/profile" ?handle-profile
|serve
```

The `new-cookie-store` creates an encrypted session store. Sessions persist user data across requests and are automatically encrypted/decrypted.

<!--



## Form data processing

This example shows comprehensive form handling, including both simple form fields and accessing all form data at once.

What's new: `full-form?` `set-content-type`

```lisp
rye .needs { http }

handle-contact: fn { w r } {
	form-data: full-form? r |^fix { 
		write w "No form data received"
		return void
	}
	
	set-content-type w "text/html"
	
	html: "<h2>Form Data Received:</h2><ul>"
	form-data |each key val {
		html: html + "<li><strong>" + key + ":</strong> " + to-string val + "</li>"
	}
	html: html + "</ul>"
	
	write w html
}

http-server ":8087"
|handle "/contact" ?handle-contact
|handle "/" "
	<form action='/contact' method='POST'>
		<input name='name' placeholder='Name' required><br>
		<input name='email' type='email' placeholder='Email' required><br>
		<textarea name='message' placeholder='Message'></textarea><br>
		<button type='submit'>Send</button>
	</form>"
|serve
```

The `full-form?` function returns all form data as a dictionary, making it easy to process multiple form fields. The `set-content-type` function ensures proper content type headers.

-->

## Static file serving with custom paths

This example demonstrates advanced static file serving with path stripping, useful for serving assets from custom URL paths.

What's new: `new-http-dir` `strip-prefix`

```lisp
rye .needs { http }

; Serve files from ./assets folder on /static/ URL path
static-handler: new-static-handler %assets |strip-prefix "/static/"

http-server ":8088"
|handle "/static/" ?static-handler
|handle "/" "
	<h1>Static File Demo</h1>
	<p>Try accessing files at <a href='/static/example.txt'>/static/example.txt</a></p>
	<img src='/static/logo.png' alt='Logo' />
	<script src='/static/app.js'></script>"
|serve
```

The `strip-prefix` function removes the URL prefix before looking up files, allowing clean separation between URL structure and file system organization.

## Handle PayPal IPN

This example demonstrates webhook handling for payment processing, including data validation, database operations, and proper error responses.

What's new: `validate` `full-form?` `sqlite`

```lisp
; HTTP server that accepts IPN requests from PayPal and stores payment info to DB
; * first it extracts data and validates it
; * if validation doesn't pass it returns a validation error to the client
; * it opens a database and creates a table if it's not yet there
; * it inserts information into the table and returns OK
;
; Code uses SQL and Validation dialects which are the usual part of Ryelang

rye |needs { sqlite http }

assure-requests-table: fn { db } {
      exec db {
           CREATE TABLE IF NOT EXISTS requests (
                  id INTEGER PRIMARY KEY AUTOINCREMENT ,
                  payment_status varchar(30) ,
                  business varchar(100) ,
                  receiver_email varchar(100) ,
                  mc_gross double ,
            )
      }
}


handle-paypal-ipn: fn { w r } {
	
	full-form? r |validate {
		payment_status: required
		business: required email
		receiver_email: required email
		mc_gross: required decimal
	}
	|^fix {
		write-header w 403 ,
		.to-string .write* w
	}
	|pass {
		db: open sqlite://paypal_ipn.db
		|assure-requests-table
	}
	|to-context .do-in {
		exec db {
            insert into requests ( id , payment_status , business , receiver_email , mc_gross  )
            values               ( null , ?payment_status , ?business , ?receiver_email , ?mc_gross )
		}
		close db
	}
	write w "OK"
}

; the server
http-server ":8080"
|handle "/paypal-ipn" ?handle-paypal-ipn
|serve
```

This example shows how to build a robust webhook handler that validates incoming data, stores it in a database, and responds appropriately to payment service notifications.
