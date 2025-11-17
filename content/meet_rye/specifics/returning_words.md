---
title: 'Returning words'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 900
summary: "Rye doesn't have a return keyword, but it has function that can also return. What?"
mygroup: true
---

_"I'll return back" Ryeminator_

## Implicit return

Like Lisps, Rebol and most functional languages Rye has an **implicit return** behaviour. We generally don't use the return "keyword" to return from function, but the value of last expression in a function is **always** returned.

Remember, everything is Rye is an expression - i.e. returns something.

```clojure
add: fn { a b } { a + b }
print add 18 24
; prints: 42

greet: fn { direction } { either direction = 'in { "Hello" } { "..." } }
print greet 'in
; prints: Hello
print greet 'out
; prints: ...

greet2 fn { dir } { switch dir { in { "Hi" } out { "Bye" } _ { "Whaa" } } }
print greet2 'in
; prints: Hi
print greet2 'out
; prints: Bye
print greet2 'up
; prints: Whaa
```

## Return function

While **rarely used** Rye has a return **function**. But how can it be a function? If you read the basics, all active components of Rye are functions and so it return. Functions if Rye can cause a return to caller and return is one of such functions.

This is not the usual way, but we could write the same thing as above using *if* and *return* functions.

```clojure
greet: fn { direction } { if direction = 'in { return "Hello" } "..." }
print greet 'in
; prints: Hello
print greet 'out
; prints: ...

find-num: fn { list n } { .for { = n |if { return true } } false }
{ 1 3 7 13 } .find-num 7
; returns 1
{ 1 3 7 13 } .find-num 8
; returns 0
```

## Returning functions

Returning words in Rye are special functions that start with a caret (`^`) symbol. They're designed to make code more readable and error handling more elegant, especially in left-to-right flow patterns.

### What Are Returning Words?

Returning words are functions that not only perform an action but also set the **return flag**, causing an immediate return from the current function. This is similar to how a `return` statement works in other languages, but integrated into Rye's functional approach.

The key characteristic of returning words is that they:
1. Execute their primary function
2. Set the return flag
3. Immediately exit the current function with their result

### Common Returning Words for Error Handling

Looking at the implementation in `builtins_base_failure.go`, we can see several important returning words:

#### ^fail

```clojure
; Creates an error, sets the failure flag, AND immediately returns from the function
^fail "Something went catastrophically wrong"
```

Unlike regular `fail` which creates an error but continues execution, `^fail` immediately exits the current function with the error.

#### ^check

```clojure
; If db/query fails, immediately return from the function with a wrapped error
db/query "SELECT * FROM users" |^check "Error querying users database"
```

The `^check` function examines if a value is in failure state and wraps it with additional context if so, then immediately returns from the function.

#### ^ensure

```clojure
; If user-age is not positive, immediately return from the function with an error
user-age > 0 |^ensure "Age must be positive"
```

The `^ensure` function validates a condition and immediately returns with an error if the condition is not met.

#### ^fix

```clojure
; If get-user fails, immediately return default-user from the current function
get-user 123 |^fix { default-user }
```

The `^fix` function handles errors by executing a block and immediately returns with the result.

## Uses of returning words

### Elegant Error Handling in Left-to-Right Flow

Returning words make error handling in left-to-right flow more elegant by allowing you to:

1. **Handle errors inline**: Instead of breaking your flow with nested if/else statements or try/catch blocks, you can handle errors right where they occur.

2. **Early return on failure**: You can immediately exit a function when an error occurs, without having to write explicit return statements or complex control flow.

3. **Chain operations with confidence**: You can build pipelines of operations where each step can validate inputs, handle errors, or exit early as needed.

### Example: API Request with Error Handling

```clojure
fetch-user-data: fn { user-id } {
  ; Validate input - return immediately if invalid
  user-id > 0 |^ensure "User ID must be positive"
  
  ; Try to fetch the user
  http/get (join "https://api.example.com/users/" user-id)
    |^check "API request failed"  ; Return immediately if request fails
    |json/parse 
    |^check "Invalid JSON response"  ; Return immediately if parsing fails
    |continue {
      ; Process the valid response
      .name |print
      .email |send-welcome-email
    }
}
```

In this example, if any step fails, the function immediately returns with an appropriate error. This creates a clean left-to-right flow where each step either succeeds and continues or fails and exits.

### Example: Database Transaction

```clojure
save-user: fn { user } {
  ; Ensure we have a valid user - return immediately if invalid
  user/valid? user |^ensure "Invalid user data"
  
  db/begin-transaction
  
  ; Use defer to ensure transaction is rolled back on any failure
  defer { db/rollback-transaction }
  
  db/insert "users" user
    |^check "Failed to insert user"  ; Return immediately if insert fails
  
  db/commit-transaction
    |^check "Failed to commit transaction"  ; Return immediately if commit fails
  
  "User saved successfully"
}
```

In this example, if any database operation fails, the function immediately returns with an error, and the deferred rollback ensures the transaction is properly handled.

### Comparison with Traditional Error Handling

Without returning words, error handling would be more verbose and less elegant:

```clojure
; Without returning words
save-user: fn { user } {
  if not user/valid? user {
    fail "Invalid user data"
    return
  }
  
  db/begin-transaction
  
  result: db/insert "users" user
  if result |failed? {
    db/rollback-transaction
    fail "Failed to insert user"
    return
  }
  
  result: db/commit-transaction
  if result |failed? {
    db/rollback-transaction
    fail "Failed to commit transaction"
    return
  }
  
  "User saved successfully"
}
```

With returning words, the same function becomes more concise and follows a cleaner left-to-right flow:

```clojure
; With returning words
save-user: fn { user } {
  user/valid? user |^ensure "Invalid user data"
  
  db/begin-transaction
  defer { db/rollback-transaction }
  
  db/insert "users" user |^check "Failed to insert user"
  db/commit-transaction |^check "Failed to commit transaction"
  
  "User saved successfully"
}
```

Returning words in Rye provide a powerful mechanism for handling errors in a left-to-right flow. By combining the action of a function with an immediate return, they allow you to write code that is both concise and robust, handling errors exactly where they occur without breaking the natural flow of your code.

This approach to error handling reflects Rye's philosophy that failures are normal, expected parts of a program's execution, and should be treated as first-class citizens that can be created, inspected, transformed, and handled with elegance.
