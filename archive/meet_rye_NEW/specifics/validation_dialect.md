---
title: 'Validation dialect'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 400
summary: ""
mygroup: true
---

## Required, optional

Validation rules begin with **required** and **optional**.

```racket
dict { "name" "jim" "score" "100" }
|validate { name: required score: optional 0 integer } |probe
; prints: #[ name: "jim" score: 100 ] 

dict { "name" "jane" }
|validate { name: required score: optional 0 integer } |probe
; prints: #[ name: "jim" score: 0 ] 

dict { }
|validate { name: required }
; returns: <Error(422): <name: required>>
```


## Types

Rye has some predefined formats / types. Like **integer**, **decimal**, **email**, **iso-date** ...

```racket
dict { "email" "abcd" "active-since" "2020" }
|validate { email: required email active-since: required iso-date }
; returns: <Error(422): <name: not email> <active-since: not iso-date>>
```

## Calc and check

To call custom code in validation dialect.

```racket
dict { "name" "jim" }
|validate { name: required calc { .capitalize } } |probe
; prints: #[ name: "Jim" ]

rules: { password: required 
	   	 check { .len >= 6 |require "shorter than 6 letters" }
		 calc { .bcrypt-hash } }
		  
parse-json '{ "password": "1234" }' |validate rules
; returns: <Error(422): <password: shorter than 6 letters>>
```

## Validating lists

Still work-in-progress but there are **some** and **any** rule words for validating lists of dicts.

```racket
parse-json '[{ "user": "jane" }, { }]'
|validate { some { user: optional "joedoe" } }
; returns: <List <Dict user: jane> <Dict user: joedoe>> 
```
