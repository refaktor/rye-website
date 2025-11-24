---
title: 'Failures and Errors'
date: 2023-04-02T09:00:00+10:00
draft: false
weight: 100
summary: "A powerful approach to error handling that treats failures as first-class citizens."
mygroup: true
---

> We're still working on this page

In most programming languages, errors are something to be feared - unwelcome interruptions that break your flow. But in Rye, failures are first-class citizens, designed to be created, inspected, transformed, and handled with elegance.

## Creating Errors

Rye offers three main ways to create errors, each with its own behavior:

```clojure
; Creates an error and sets the failure flag, but continues execution
fail "Something went wrong"

; Creates an error, sets the failure flag, AND immediately returns from the function
^fail "Something went catastrophically wrong"

; Just creates an error object without setting any flags
err: failure "This is just an error value"
```

The difference is subtle but powerful. `fail` signals a problem but lets the surrounding code decide what to do, while `^fail` is more decisive, immediately exiting the current function. And `failure` just creates an error value without affecting program flow at all.

## Anatomy of an Error

Errors in Rye aren't just strings - they're rich objects that can carry structured information:

```clojure
; Create an error with a status code
api-error: fail 404

; Create an error with a detailed message
validation-error: fail "Invalid email format"

; Create an error with both code and additional details
db-error: fail { 
  "code" 1001 
  "table" "users" 
  "operation" "insert" 
}
```

You can then inspect these errors using accessor functions:

```clojure
api-error |status?    ; Returns 404
validation-error |message?  ; Returns "Invalid email format"
db-error |details? .table  ; Returns "users"
```

## Error Handling Combinators

Where Rye really shines is in how it lets you handle errors. Let's look at some patterns:

### The Fix Pattern

The `fix` combinator lets you handle errors by providing a recovery block:

```clojure
; Try to get a user, or return a default if it fails
get-user 123 |fix { default-user }

; Try to parse a date, or return today if it fails
parse-date user-input |fix { today }
```

The `^fix` variant also sets the return flag, immediately exiting the current function with the result of the handler:

```clojure
; If get-user fails, immediately return default-user from the current function
get-user 123 |^fix { default-user }
```

### The Check Pattern

The `check` combinator lets you transform errors by wrapping them with additional context:

```clojure
; Add context to any error that might occur
db/query "SELECT * FROM users" |check "Error querying users database"

; In a chain of operations, add context at each step
file/read "config.json" 
  |^check "Failed to read config file" 
  |json/parse 
  |^check "Config file contains invalid JSON"
```

The `^check` variant also sets the return flag, immediately exiting the current function with the wrapped error:

```clojure
; If db/query fails, immediately return from the function with a wrapped error
db/query  "SELECT * FROM users" |^check "Error querying users database"
```

### The Ensure Pattern

The `ensure` combinator lets you validate conditions and fail if they're not met:

```clojure
; Validate user input
user-age > 0 |ensure "Age must be positive"
user-email .contains "@" |ensure "Email must contain @"

; Check preconditions before proceeding
db/connected? .ensure "Database connection required"
user/has-permission? 'admin |ensure "Admin permission required"
```

The `^ensure` variant also sets the return flag, immediately exiting the current function if the condition is not met:

```clojure
; If user-age is not positive, immediately return from the function with an error
user-age > 0 |^ensure "Age must be positive"
```

### The Continue Pattern

The `continue` combinator is the opposite of `fix` - it executes a block only if the value is not in failure state:

```clojure
; Process the result only if the operation succeeded
get-user 123 |continue { 
  .name |print 
  .email |send-welcome-email
}
```

### The Fix\Continue Pattern

## Advanced Error Handling

Rye provides several advanced error handling mechanisms:

### Retry

The `retry` combinator lets you automatically retry an operation a specified number of times:

```clojure
; Try the HTTP request up to 3 times before giving up
retry 3 {
  http/get "https://api.example.com/data"
}
```

### Timeout

The `timeout` combinator lets you set a maximum execution time for an operation:

```clojure
; Fail if the database query takes more than 5 seconds
timeout 5000 {
  db/query "SELECT * FROM large_table WHERE complex_condition"
}
```

### Disarming Errors

The `disarm` function clears the failure flag while preserving the error object, allowing you to inspect an error without propagating it:

```clojure
; Check if an operation failed without propagating the error
result: operation |disarm
if result |failed? [
  log/error result |message?
]
```

### Error Testing

The `failed?` function tests if a value is an error:

```
```

## Real-World Examples

Let's look at some practical examples of how these patterns come together:

### Example 1: API Request with Error Handling

```clojure
fetch-user-data: fn { user-id } {
  ; Validate input
  user-id > 0 |^ensure "User ID must be positive"
  
  ; Try to fetch the user, with multiple layers of error handling
  Get https://api.example.com/users/ ++ user-id
    |check "API request failed" 
    |parse-json 
    |check "Invalid JSON response"
    |fix { 
      ; If anything failed, return a default user
      { "id" user-id "name" "Unknown" "status" "error" }
    }
}
```

### Example 2: Database Transaction with Retry

```clojure
```

### Example 3: Cascading Error Handling

```clojure
```
