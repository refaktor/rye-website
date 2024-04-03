---
title: 'Rye structures'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 200
summary: "And combine those into bigger data structures."
---

Rye code is first and foremost a data format - a way to define and store data. Because of its symmetry (everything is equal to everything else), and many value types it can elegantly define all kinds of data. 

## JSON like

Rye code can be used to define structures that JSON can define.

```json
[
  { "name": "Jim",  "score": 230, "email": "jim@json.org"  }
  { "name": "Anne", "score": 290, "email": "anne@json.org" }
]
```
Something similar in Rye:

```clojure
{
  { name: "Jim"   score: 230  email: jim@json.org  }
  { name: "Anne"  score: 290  email: anne@json.org }
}
```

Rye has many more types available than JSON. In inner blocks above, we used: 

```clojure
name:         ; set-word
"Jim"         ; string
230           ; integer 
jim@json.org  ; email
```

## XML like

XML nodes have default data, and also attributes.

```xml
<users current="1">
  <user id="1">jim@ryelang.org</user>
  <user id="2">anne@ryelang.org</user>
</users>
```

We could define this information in something like this, for example:

```clojure
{ users current: 1 data: { 
  { user id: 1 data: jim@ryelang.org  }
  { user id: 2 data: anne@ryelang.org  }
} }
```

## List of Logo Turtle commands

With Rye we often ask ourselves how would we represent the subject in our code, and we have a lot of freedom to do so. Let's say we have a logo turtle, that can rotate and move in the direction it's turned.
We could represent the commands with this valid Rye.

```clojure
{ 
  pen 'down 
  move 100 rotate 90 
  move 100 rotate 90
  move 100 rotate 90
  move 100
  pen 'up
} 
```

BTW, **newlines have no special meaning in Rye**. They are just spacing

---
-- Bonus --

## Validation rules

How would we best represent the validation rules for some input information. Below is one example.

```clojure
{ 
  name: required string calculate { .capitalize }
  birth: required iso-date check { > now\year } "Can't be born in the future"
  score: optional 50 integer
} 
```

We're more used that the structure or keys determine information. In Rye, with its many value and word types, the types themselves can sometimes define the information structure. 

Here a set-word (like _name:_) marks the field name **and** the start of the rules for it. We could pack each ruleset in a block, but it would 


## A NPC dialogue

We could define a dialogue options for specific NPC in an imaginary RPG game for example like this:

```clojure
{ 
  0.3 { "I'm busy" }
  0.7 { "How can I help you?" options {
      "Where could I find the Pub" {
        0.4 { "Go to the church and turn right." do { mark-on-map 'church } }
        0.6 { "I hate early drinkers." }
      }
      "I'm so tired ..." { 
        1.0 { "Here, have and apple!" do { add-health 25 } }
      }
    }
  }
}
```



## A greeter NPC

Let's imagine rules for another, more direct NPC that stands in front of the hotel and greets the player in the morning and evening:

```clojure
if hour < 8 { say "Good morning, sire!" }

if hour > 18 { say "Good evening, mr.!" }
```

Well, if **hour** and **say** are functions in our context, this above is just regular **Rye language code**. The same goes for Logo Turtle example, if we make functions _pen_, _move_ and _rotate_ each taking one argument, we can evaluate it as normal Rye. And the NPC dialogue has Rye code in its _do_ blocks.

