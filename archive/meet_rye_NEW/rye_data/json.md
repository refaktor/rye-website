---
title: 'Rye as JSON and more'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 100
summary: ""
mygroup: true
---

## Is Rye like JSON

Rye code is very similar to JSON and can be used to define similar things.

```json
[
  { "name": "Jim",  "score": 230, "email": "jim@json.org"  }
  { "name": "Anne", "score": 290, "email": "anne@json.org" }
]
```
In Rye we can create something similar:

```racket
{
  { name: "Jim"   score: 230  email: jim@json.org  }
  { name: "Anne"  score: 290  email: anne@json.org }
}
```

Rye has more types available than JSON. In our inner blocks we used: 

```racket
name:         ; set-word
"Jim"         ; string
230           ; integer 
jim@json.org  ; email
```

---

## Bonus: logo turtle

With Rye we often ask ourself. How would I define, declare this type of problem, and we have a lot of freedom to do so. Let's say we have a logo turtle, that can rotate and move in the direction it's turned.

```racket
{ 
  pen 'down 
  move 100 rotate 90 
  move 100 rotate 90
  move 100 rotate 90
  move 100
  pen 'up
} 
```

## A NPC



```racket
{ 
  { "How are you today" }
  { "Do you need anything" 
    'Yes 
	'No
  }
} ```
