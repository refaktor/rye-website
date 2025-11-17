---
title: 'Validation Dialect'
date: 2023-04-02T08:00:00+10:00
draft: false
weight: 200
summary: "A powerful dialect for validating and transforming data in Rye."
mygroup: true
---

Data validation is a critical aspect of any robust application. Rye provides a powerful validation dialect that allows you to validate and transform data according to specified rules.

## Basic Validation

The validation dialect in Rye is centered around the `validate` function, which takes two arguments: the data to validate (usually a dictionary) and a block of validation rules.

```clojure
validate dict { name: "John" age: 30 } {
    name: required
    age: required integer
}
; returns: dict { name: "John" age: 30 }
```

When validation succeeds, it returns a dictionary with the validated and potentially transformed values. If validation fails, it returns an error.

## Validation Rules

### Required and Optional Values

The most basic validation rules are `required` and `optional`:

```clojure
; Ensure a value is present
validate dict { name: "John" } { name: required }
; returns: dict { name: "John" }

; Validation fails if a required value is missing
validate dict { } { name: required }
; returns: error with status 403 and message "validation error"

; Provide a default value if not present
validate dict { } { name: optional "Anonymous" }
; returns: dict { name: "Anonymous" }

; Use both required and optional fields
validate dict { name: "John" } { 
    name: required 
    age: optional 30 
}
; returns: dict { name: "John" age: 30 }
```

### Type Validation and Conversion

The validation dialect can validate and convert values to specific types:

```clojure
; Convert string to integer
validate dict { age: "30" } { age: required integer }
; returns: dict { age: 30 }

; Convert to decimal
validate dict { price: "19.99" } { price: required decimal }
; returns: dict { price: 19.99 }

; Ensure value is a string
validate dict { name: "John" } { name: required string }
; returns: dict { name: "John" }

; Validate email format
validate dict { email: "john@example.com" } { email: required email }
; returns: dict { email: "john@example.com" }

; Validate and convert date format
validate dict { birthdate: "30.12.2024" } { birthdate: required date }
; returns: dict with birthdate as a date object
```

### Conditional Validation with Check

You can use the `check` rule to validate a value against a condition:

```clojure
; Ensure age is less than 100
validate dict { age: 30 } { age: required integer check { < 100 } }
; returns: dict { age: 30 }

; Validation fails if condition is not met
validate dict { age: 150 } { age: required integer check { < 100 } }
; returns: error
```

### Transforming Values with Calc

The `calc` rule allows you to transform a value using a calculation:

```clojure
; Add 10 to the age
validate dict { age: 30 } { age: required integer calc { + 10 } }
; returns: dict { age: 40 }

; Convert temperature from Celsius to Fahrenheit
validate dict { temp_c: 25 } { temp_c: required decimal calc { * 1.8 + 32 } }
; returns: dict { temp_c: 77.0 }
```

### Validating Lists with Some

For lists, you can use the `some` rule to apply validation to each item:

```clojure
; Validate a list of integers
validate list [1 "2" 3] { some { required integer } }
; returns: list [1 2 3]
```

## Error Handling

When validation fails, it returns an error with status 403 and a message "validation error". You can use error handling functions to get more details:

```clojure
; Get the error status
validate dict { email: "not-an-email" } { email: required email } |disarm |status?
; returns: 403

; Get the error message
validate dict { email: "not-an-email" } { email: required email } |disarm |message?
; returns: "validation error"

; Get detailed error information
validate dict { email: "not-an-email" } { email: required email } |disarm |details?
; returns: dict { email: "not email" }
```

## Context Objects for Easy Access

The `validate>ctx` function works like `validate` but returns a context object for easier field access:

```clojure
ctx: validate>ctx dict { name: "John" age: 30 } {
    name: required
    age: required integer
}

; Access fields directly
ctx -> 'name
; returns: "John"

ctx -> 'age
; returns: 30
```

## Combining Rules

You can combine multiple validation rules for more complex validations:

```clojure
validate dict { 
    name: "John" 
    age: "30" 
    email: "john@example.com" 
} {
    name: required string
    age: required integer check { > 0 check { < 120 } }
    email: required email
}
; returns: validated dictionary with converted values
```

## Practical Example

Here's a more complete example showing how validation might be used in a web application:

```clojure
; Define a function to handle user registration
register-user: func [request] [
    ; Validate the request data
    user-data: validate>ctx request/body {
        username: required string
        email: required email
        password: required string
        age: optional 0 integer check { >= 18 }
    }
    
    ; If validation succeeded, proceed with registration
    if error? user-data [
        return response 400 user-data
    ]
    
    ; Use the validated data
    create-user user-data
    response 201 "User registered successfully"
]
```

The validation dialect provides a powerful yet concise way to ensure your data meets your requirements, with automatic type conversion and detailed error reporting when validation fails.
