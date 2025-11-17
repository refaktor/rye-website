---
title: 'Limiting Side Effects'
date: 2023-04-02T09:30:00+10:00
draft: false
weight: 200
summary: "How Rye provides certainty and safety by strictly controlling state changes."
mygroup: true
---

One of Rye's core design principles is to be strict about state changes. By limiting how and where side effects can occur, Rye programs become more predictable, easier to reason about, and less prone to bugs. This document explains how Rye achieves this through various language features.

## Expressions Over Statements

In Rye, everything is an expression that returns a value. Unlike many imperative languages that rely on statements that modify state, Rye encourages a more functional approach where functions transform values rather than modify them.

```clojure
; In many languages, you might do:
x = 5
x = x + 10  ; Modifying x in place

; In Rye, you can use the var function:
var 'x 5
x:: x + 10  ; Creating a new value and rebinding x

; or mod-words:
x:: 5
x:: x + 10  ; Creating a new value and rebinding x

; using just set-words (used generally to assign values to words)
; won't work for two reasons. The word is a constant and
; you can't use set-word to change a value of a word, even if it'a
; a variable
x: 5
x: x + 10   ; error
```

This expression-oriented approach makes code more predictable and easier to reason about, as you're always dealing with transformations rather than mutations.

## Immutable by Default

Most Rye values are immutable by default, and most functions return new values rather than modifying existing ones. This immutability provides certainty about your data and eliminates a whole class of bugs related to unexpected mutations.

```clojure
; Creating a list
names: { "Alice" "Bob" "Charlie" }

; Adding an element returns a new list, doesn't modify the original
new-names: names ++ "David"

; Original list remains unchanged
names |print      ; Prints: { "Alice" "Bob" "Charlie" }
new-names |print  ; Prints: { "Alice" "Bob" "Charlie" "David" }
```

This approach encourages you to think in terms of data transformations rather than state mutations, leading to cleaner, more predictable code.

## Explicit Mutation with !

When you do need to modify values in place (perhaps for performance reasons), Rye makes this explicit by requiring functions that mutate state to end with an exclamation mark (!). This visual indicator makes side effects immediately obvious in your code.

```clojure
; Creating a reference to a mutable list
vat 'names ref { "Alice" "Bob" "Charlie" }

; Modifying the list in place
append! names "David"

; The original reference now contains the modified list
names |print
; Prints: { "Alice" "Bob" "Charlie" "David" }
```

Other examples of functions that modify state in place include:

```clojure
; Incrementing a number in place
var 'count: 0
inc! count  ; count is now 1

; Changing a value in a dictionary
user:: ref dict { "name" "Alice" "age" 30 }
mod! 'user "age" 31                 ; user now has age 31

; Removing an element from a block
items: ref { "apple" "banana" "cherry" }
remove! items "banana"              ; items is now { "apple" "cherry" }
```

By making mutations explicit with the ! suffix, Rye code clearly communicates where side effects occur, making it easier to track state changes and reason about program behavior.

## Constants vs Variables

Rye makes a clear distinction between constants (values that cannot be changed) and variables (values that can be changed). By default, when you use a set-word (word:), you're creating a constant that cannot be overwritten.

```clojure
; Setting a constant
pi: 3.14159

; Trying to overwrite it will cause an error
pi: 3.14  ; Error: Cannot redefine word 'pi'
```

This prevents accidental redefinition of important values and makes your code more predictable. If you need a value that can change, you have two options:

### 1. Using Mod-Words

Mod-words (word::) allow you to modify an existing binding. They are visually distinct from set-words, making it clear when a value is being changed.

```clojure
; Setting an initial value
counter:: 0

; Modifying it with a mod-word
counter:: counter + 1  ; counter is now 1
counter:: counter + 1  ; counter is now 2
```

### 2. Declaring Variables

Alternatively, you can explicitly declare a word as a variable using the `var` function, which allows it to be modified later.

```clojure
; Declaring a variable
var 'name "Jim"

; Now it can be modified with a mod-word
name:: "James"  ; name is now "James"
```

By requiring explicit declaration of variables and using visually distinct syntax for modifications, Rye makes state changes more obvious and intentional.

## Context Isolation

Rye strictly limits the scope of state changes to the current context. You cannot directly modify values in parent or sub-contexts, which prevents unexpected side effects across different parts of your program.

```clojure
; Creating a context with some values
user-ctx: context {
    name: "Alice"
    age:: 30
}

; You cannot directly modify values in another context
user-ctx/age:: 31  ; Error: Cannot modify value in another context

; Instead, you must use functions that operate within that context
update-age: func [ctx new-age] [
    do/in ctx [
        age:: new-age  ; This works because we're inside the context
    ]
]

; Now you can update the age properly
update-age user-ctx 31
```

Functions that modify state in other contexts should also follow the ! naming convention:

```clojure
update-age!: func [ctx new-age] [
    do/in ctx [
        age:: new-age
    ]
    ctx  ; Return the modified context
]
```

This context isolation ensures that state changes are localized and predictable, making it easier to reason about your code and avoid unexpected side effects.

## Real-World Example

Let's look at a more complete example that demonstrates these principles in action:

```clojure
; Define a function to process an order
process-order: func [order] [
    ; Create a new order with validated fields
    validated-order: validate order {
        customer-id: required integer
        items: required
        total: required decimal
    }
    
    ; Process payment (returns a new order with payment info)
    paid-order: process-payment validated-order
    
    ; Update inventory (note the ! indicating state change)
    update-inventory! paid-order/items
    
    ; Return the final order with status
    paid-order ++ { "status" "completed" }
]

; Using the function
order: {
    "customer-id" "1001"
    "items" { { "id" "A123" "qty" 2 } { "id" "B456" "qty" 1 } }
    "total" "99.95"
}

; Process the order (note that the original order is unchanged)
result: process-order order
```

In this example:
- The original order is not modified
- The function returns a new order with additional information
- The only side effect is explicitly marked with the ! suffix
- All transformations are expressed as data flowing through functions

## Benefits of Limiting Side Effects

By being strict about state changes, Rye provides several important benefits:

1. **Predictability**: When side effects are limited and explicit, code behavior becomes more predictable.

2. **Testability**: Pure functions with no side effects are much easier to test than functions with hidden mutations.

3. **Reasoning**: It's easier to reason about code when you can clearly see where state changes occur.

4. **Concurrency**: Code with fewer side effects is naturally more amenable to concurrent execution.

5. **Debugging**: When bugs occur, it's easier to track down the source when state changes are explicit and limited.

These benefits lead to more robust, maintainable code that's less prone to the kinds of subtle bugs that plague many software systems.
