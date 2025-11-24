Sure! Here’s the revised version of the Markdown, along with a list of changes made for clarity and correctness.

---

title: 'Structures'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 200
summary: "And combine those into more versatile structures."
---

_"Stone upon stone, a palace - value upon value, a structure."_

## A List

Using the value types we saw on the previous page, we can construct various more complex structures, the simplest of which is a list.

We use *block* and *email* value types to construct this list of important emails:

```clojure
{ 
    steve@microple.com 
    jeff@aplezon.com 
    bill@amazosoft.com 
}
```

## JSON

Rye can recreate a structure typical for JSON. An array of JSON objects could look like this:

```json
[
  { "name": "Jim",  "score": 230, "email": "jim@json.org" },
  { "name": "Anne", "score": 290, "email": "anne@json.org" }
]
```

This could be represented in Rye as follows. Be aware that second-level blocks are still just blocks. Rye doesn't have special syntax for more specific collection types like dictionaries, but it does include dictionaries and a function that turns blocks like the one below into dictionaries.

```clojure
{
    { name: "Jim"      score: 230  email: jim@json.org   }
    { name: "Anne"     score: 290  email: anne@json.org  }
}
```

Otherwise, Rye has more value types available than JSON. Here we used:

```clojure
{ }           ; block
name:         ; set-word
"Jim"         ; string
230           ; integer 
jim@json.org  ; email
```

## XML

XML nodes contain a number of attributes as well as main data or content.

```xml
<users current="1">
    <user id="1">jim@ryelang.org</user>
    <user id="2">anne@ryelang.org</user>
</users>
```

In Rye, we could represent this as:

```clojure
{ users current: 1 data: { 
    { user id: 1 data: jim@ryelang.org }
    { user id: 2 data: anne@ryelang.org }
} }
```

## Logo Turtle Commands

With Rye, we often ask ourselves how best to represent problems in our code, and we have a lot of freedom to do so.

Let's say we have a Logo turtle that can rotate and move in the direction it is turned. We could represent a series of commands like so:

```clojure
{ 
    pen 'down 
    move 100
    rotate 90 
    move 100 
    rotate 90 
    move 100 
    rotate 90 
    move 100 
    pen 'up
} 
```

Here we use words like `pen`, `move`, and `rotate`, along with literal words like `'down` and `'up`, and integers. Newlines have no special meaning in Rye; they are merely for spacing.

## Validation Rules

How would we best represent the validation rules for some input information? Below is a validation dialect that is part of base Rye:

```clojure
{ 
    name: required string calculate { .capitalize }
    birth: required iso-date check { > now\year } "Can't be born in the future"
    score: optional 50 integer
} 
```

<!-- We're more accustomed to having the structure or keys determine information. In Rye, with its many value and word types, the types themselves can sometimes define the information structure. 

Here a set-word (like _name:_) marks the field name **and** the start of the rules for it. While we could pack each ruleset in a block, it would be less clear. -->

## An NPC Dialogue

We could define a dialogue tree with desired probabilities for a specific NPC in an imaginary RPG game, for example, like this:

```clojure
{ 
    0.3 { "I'm busy." }
    0.7 { "How can I help you?" options {
        "Where could I find the Pub?" {
            0.4 { "Go to the church and turn right." do { mark-on-map 'church } }
            0.6 { "I hate early drinkers." }
        }
        "I'm so tired..." { 
            1.0 { "Here, have an apple!" do { add-health 25 } }
        }
    }}
}
```

## A Greeter NPC

Let's imagine rules for another, more direct NPC that stands in front of the hotel and greets the player in the morning and evening:

```clojure
{ 
    if hour < 8 { say "Good morning, sire!" }
    if hour > 18 { say "Good evening, sir!" }
}
```

Again, if `hour` and `say` are functions in our context, the above is just regular **Rye code**. 

The same goes for the **Logo Turtle** example above—if we define functions `pen`, `move`, and `rotate`, each taking one argument, we can evaluate it as normal Rye. Note that the NPC dialog example has Rye code in its **do** blocks.

---

### Changes Made:

1. Corrected capitalization and punctuation for consistency and clarity (e.g., "Stone upon stone, a palace").
2. Clarified the descriptions and examples for better understanding.
3. Added missing commas in JSON representation for proper syntax.
4. Fixed formatting in the Logo Turtle section for improved readability.
5. Added a period at the end of some sentences for grammatical consistency.
6. Clarified ambiguous phrases for better comprehension.
7. Updated "sir." in the greeter NPC code for better sentence structure.
8. Corrected "have and apple!" to "have an apple!" in the NPC dialogue example for grammatical accuracy.
9. Added clarity to the final comments within the validation rules section to improve understanding.