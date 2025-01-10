---
title: 'Structures'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 200
summary: "And combine those into more versatile structures."
---

_"stone upon stona, a palace - value upon value, a structure"_

## A list

Using value types we saw in the previous page, we can construct various more complex structures. The simplest being a list.

We use *block* and *email* value types to construct this list of important emails:

```clojure
{ 
	steve@microple.com 
	jeff@aplezon.com 
	bill@amazosoft.com 
}
```

## JSON

Rye can recreate a structure typical for JSON. And array of JSON objects:

```json
[
  { "name": "Jim",  "score": 230, "email": "jim@json.org"  }
  { "name": "Anne", "score": 290, "email": "anne@json.org" }
]
```

Could be something like this in Rye.

```clojure
{
  { name: "Jim"   score: 230  email: jim@json.org  }
  { name: "Anne"  score: 290  email: anne@json.org }
}
```

Rye has more available value types than JSON. Here we used:

```clojure
{ }           ; block
name:         ; set-word
"Jim"         ; string
230           ; integer 
jim@json.org  ; email
```

## XML

XML nodes can have data and also attributes.

```xml
<users current="1">
  <user id="1">jim@ryelang.org</user>
  <user id="2">anne@ryelang.org</user>
</users>
```

In Rye we could do something like this:

```clojure
{ users current: 1 data: { 
  { user id: 1 data: jim@ryelang.org  }
  { user id: 2 data: anne@ryelang.org  }
} }
```

## Logo Turtle commands

With Rye we often ask ourselves how would we represent the problem in our code, and we have a lot of freedom to do so. 

Let's say we have a logo turtle, that can rotate and move in the direction it's turned. We could represent commands like this:

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

Newlines have no special meaning in Rye. They are just spacing.

## Validation rules

How would we best represent the validation rules for some input information. Below is one example.

```clojure
{ 
  name: required string calculate { .capitalize }
  birth: required iso-date check { > now\year } "Can't be born in the future"
  score: optional 50 integer
} 
```

<!-- We're more used that the structure or keys determine information. In Rye, with its many value and word types, the types themselves can sometimes define the information structure. 

Here a set-word (like _name:_) marks the field name **and** the start of the rules for it. We could pack each ruleset in a block, but it would  -->


## A NPC dialogue

We could define dialog options for a specific NPC in an imaginary RPG game for example like this:

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

Well, if **hour** and **say** are functions in our context, the above is just regular **Rye language code**. The same goes for the Logo Turtle example above, if we make functions _pen_, _move_ and _rotate_ each taking one argument, we can evaluate it as normal Rye. Note that the NPC dialog example has Rye code in its _do_ blocks.

