---
title: "Serving HTTP"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 50
summary: "An growing list of server-side examples."
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


## Hello world over HTTP

What's new: `http-server` `handle` `write` `serve`

```lisp
rye .needs { http }

http-server ":8081"
|handle "/hello" fn { w r } { .write "Hello world" }
|serve

; or 
; |handle "/hello" "Hello world"
```

## Serve folder over HTTP

Keywords: `http` `static` `folder`

```lisp
rye .needs { http }

http-server ":8085"
|handle "/" new-static-handler %public_html
|serve
```

## Serve load averages over HTTP

What's new: `http` `os` `load-avg?` `to-json` 

```lisp
rye .needs { http os }

http-server ":8086"
|handle "/" fn { w r } { do\par os { load-avg? |to-json |write* w } }
|serve
```

## Websocket server

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

## Controlled file download

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



## Handle file uploads

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

## Handle PayPal IPN

What's new: `validate` `full-form?` `sqlite`

```lisp
; HTTP server that accepts IPN requests from PayPal and stores payment info to DB
; * first it extracts data and validates it
; * if validation doesn't pass it returns a validation error to the client
; * it opens a database and creates a table if it's not yet there
; * it inserts information into the table and retursn OK
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
