# Why Use Rye: Key Features and Benefits

## Core Reasons to Use Rye

1. **Safety-First Programming**
   - Explicit state changes with the `!` suffix for functions that modify state
   - Immutable values by default, reducing unexpected side effects
   - Clear distinction between constants and variables
   - Context isolation to prevent unexpected state changes across different parts of a program
   - Validation dialect for robust data validation and transformation

   ```clojure
   ; Explicit mutation with ! suffix
   counter: ref 0
   append! counter 1  ; counter is now 1
   
   ; Immutable by default
   names: { "Alice" "Bob" "Charlie" }
   new-names: names + "David"  ; Original list unchanged
   names |print  ; Prints: { "Alice" "Bob" "Charlie" }
   ```

2. **Elegant Error Handling**
   - First-class failure handling with `fail`, `^fail`, and `failure`
   - Composable error handling with `fix`, `check`, and `ensure` combinators
   - Returning words (`^`) for elegant early returns in left-to-right flow
   - More expressive and concise than traditional try/catch blocks

   ```clojure
   ; Using fix to handle errors elegantly
   read %config.json |fix { 
     ; If file read fails, use default config
     { "theme" "default" "language" "en" } 
   } |json/parse
   
   ; Using ensure to validate conditions
   user-age > 18 |ensure "Must be an adult to proceed"
   ```

3. **Flexible Syntax for Data Flow**
   - Op-words (`.word`) and pipe-words (`|word`) for left-to-right code flow
   - Expression-based language where everything returns a value
   - Homoiconic structure (code is data, data is code)
   - Function-oriented approach with no keywords, only function calls

   ```clojure
   ; Using pipe-words for data flow
   "Hello World" |replace "World" "Rye" |to-upper |print
   ; Prints: HELLO RYE
   
   ; Using op-words for method-like calls
   "Hello World" .replace "World" "Rye" .to-upper .print
   ; Prints: HELLO RYE
   ```

4. **Security Features**
   - Seccomp for system call filtering
   - Landlock for filesystem access control
   - Code signing verification for trusted script execution
   - Explicit separation of pure and impure functions

   ```clojure
   ; Running with seccomp profile for read-only operations
   ; $ rye --seccomp-profile=readonly script.rye
   
   ; Pure functions are marked as such
   sum-squares: fn { a b } { 
     a * a + b * b 
   } |mark-pure
   
   ; This will fail in readonly mode
   write %file.txt "Hello"
   ```

5. **Integration with Go**
   - Excellent scripting companion for Go programs
   - Can be embedded into Go applications as a scripting or configuration language
   - Go-based interpreter for performance

   ```go
   // Embedding Rye in a Go application
   package main

   import (
     "github.com/refaktor/rye/env"
     "github.com/refaktor/rye/evaldo"
   )

   func main() {
     ps := env.NewProgramState()
     evaldo.EvalString(ps, `print "Hello from embedded Rye!"`)
   }
   ```

6. **Multiple Dialects**
   - Specialized interpreters for different tasks (rye, eyr, math)
   - Validation dialect for data validation
   - Ability to create custom dialects for specific domains

   ```clojure
   ; Using the math dialect for mathematical expressions
   ; $ rye -lang math
   x> 10 * 5 + 2 * 3
   ; Returns: 56 (follows standard math precedence)
   
   ; Using the stack-based Eyr dialect
   ; $ rye -lang eyr
   x> 10 5 * 2 3 * +
   ; Returns: 56 (postfix notation)
   ```

7. **Rich Data Processing**
   - Built-in support for working with CSV, JSON, and other data formats
   - Powerful data transformation capabilities
   - Spreadsheet-like operations with tables

   ```clojure
   ; Loading and transforming CSV data
   load\csv %users.csv
     |where-greater "age" 18
     |sort-by "last_name"
     |select { "first_name" "last_name" "email" }
     |save\csv %adult-users.csv
   
   ; Working with JSON
   read %config.json |json/parse
     |change "theme" "dark"
     |json/stringify
     |write %config.json
   ```

8. **Modern Language Model Integration**
   - Model Context Protocol (MCP) for communication between language models and external tools
   - Create MCP servers, clients, resources, tools, and prompts

   ```clojure
   ; Creating an MCP server
   server: mcp-server//create "weather-api" "1.0.0"
   
   ; Adding a tool to the server
   tool: mcp//create-tool "get-weather" "Gets weather for a location"
   
   ; Implementing the tool handler
   tool .set-handler fn { args } {
     location: args .get "location"
     { "temperature" 22 "condition" "sunny" "location" location }
   }
   
   ; Adding the tool to the server
   server .add-tool tool
   ```

## Features to Focus On

1. **Pipe-Based Data Flow**
   - The pipe operator (`|`) for creating clean data transformation pipelines
   - Op-words (`.word`) for object-oriented style method calls
   - Chaining operations for concise and readable code

   ```clojure
   ; Processing a list of numbers with pipes
   { 1 2 3 4 5 6 7 8 9 10 }
     |filter { > 5 }
     |map { * 2 }
     |sum
     |print
   ; Prints: 90
   
   ; Same operations with op-words
   { 1 2 3 4 5 6 7 8 9 10 }
     .filter { > 5 }
     .map { * 2 }
     .sum
     .print
   ; Prints: 90
   ```

2. **Failure Handling System**
   - The `fix` pattern for elegant error recovery
   - The `check` pattern for adding context to errors
   - The `ensure` pattern for validating conditions
   - Returning words (`^`) for early returns in error cases

   ```clojure
   ; Elegant error handling with fix
   process-order: fn { order-id } {
     ; Try to get the order, or return "not found" if it fails
     db/get-order order-id |fix { "Order not found" }
     
     ; Add context to any error that might occur
     |check "Error processing order"
     
     ; Validate that the order is valid
     |ensure { .status = 'pending } "Order is not pending"
     
     ; Process the order
     |process-payment
     |update-inventory
     |send-confirmation-email
   }
   ```

3. **Validation Dialect**
   - Type validation and conversion
   - Required and optional fields
   - Default values
   - Conditional validation with `check`
   - Value transformation with `calc`

   ```clojure
   ; Validating user input
   validate user-input {
     username: required string
     email: required email
     age: required integer check { >= 18 }
     role: optional "user"
     score: required decimal calc { * 1.5 }
   }
   
   ; If validation succeeds, you get a dictionary with validated values
   ; If it fails, you get an error with details about what failed
   ```

4. **Security Features**
   - Seccomp profiles for limiting system calls
   - Landlock for filesystem access control
   - Code signing for script verification

   ```clojure
   ; Running a script with landlock to restrict filesystem access
   ; $ rye --landlock --landlock-profile=readonly --landlock-paths=/data script.rye
   
   ; This will succeed because /data is allowed
   read %/data/config.json
   
   ; This will fail because /etc is not allowed
   read %/etc/passwd
   
   ; Verifying a signed script
   ; $ rye --codesig script.rye
   ```

5. **Immutability and Explicit State Changes**
   - Immutable values by default
   - Explicit mutation with `!` suffix
   - Clear distinction between constants and variables
   - Context isolation for controlled state changes

   ```clojure
   ; Immutable operations create new values
   list: { 1 2 3 }
   doubled: list |map { * 2 }  ; Creates a new list
   list |print      ; Prints: { 1 2 3 }
   doubled |print   ; Prints: { 2 4 6 }
   
   ; Mutable operations modify in place
   mutable-list: ref { 1 2 3 }
   map! mutable-list { * 2 }  ; Modifies the list in place
   mutable-list |print  ; Prints: { 2 4 6 }
   ```

6. **Multiple Dialects and Extensibility**
   - Different language modes for different tasks
   - Ability to create custom dialects
   - Extensible with new functions and types

   ```clojure
   ; Creating a custom dialect for configuration
   config-dialect: dialect {
     ; Define custom words for the dialect
     set-option: fn { key value } { 
       options .change key value 
     }
     
     get-option: fn { key } { 
       options .get key 
     }
     
     ; Define the evaluator function
     evaluator: fn { block } {
       options: dict []
       eval-block block
       options
     }
   }
   
   ; Using the custom dialect
   config: config-dialect {
     set-option "theme" "dark"
     set-option "language" "en"
     set-option "notifications" true
   }
   ```

7. **Go Integration**
   - Embedding Rye in Go applications
   - Using Rye as a configuration language
   - Extending Rye with Go functions

   ```go
   // Adding a custom function to Rye from Go
   package main

   import (
     "github.com/refaktor/rye/env"
     "github.com/refaktor/rye/evaldo"
   )

   func main() {
     ps := env.NewProgramState()
     
     // Register a custom function
     evaldo.RegisterBuiltin(ps, "greet", 1, func(ps *env.ProgramState, args ...env.Object) env.Object {
       name := args[0].(env.String).Value
       return *env.NewString("Hello, " + name + "!")
     })
     
     // Use the custom function
     evaldo.EvalString(ps, `greet "World" |print`)
     // Prints: Hello, World!
   }
   ```

8. **Interactive Development**
   - REPL console for interactive development
   - Save and continue functionality
   - Various command-line options for flexibility

   ```bash
   # Start the REPL
   $ rye
   x> 21 * 2
   42
   
   # Define a function
   x> double: fn { x } { x * 2 }
   x> double 21
   42
   
   # Save the current state
   x> save\current
   # saved to console_2025_04_24_1150.rye
   
   # Later, continue from where you left off
   $ rye cont
   x> double 21
   42
   ```

## Ideal Use Cases

1. **Data Processing and Transformation**
   - ETL (Extract, Transform, Load) operations
   - Data validation and cleaning
   - Working with CSV, JSON, and other data formats

   ```clojure
   ; Extract data from CSV, transform it, and load it into a database
   load\csv %sales.csv
     |where-greater "amount" 1000
     |group-by "region"
     |map { [ .key .value |sum-by "amount" ] }
     |sort-by 1 'desc
     |save\csv %sales-by-region.csv
   ```

2. **Configuration and Scripting**
   - Advanced configuration files with logic
   - System automation scripts
   - Embedding as a scripting language in Go applications

   ```clojure
   ; Dynamic configuration with logic
   config: {
     "environment" env/get "APP_ENV" |fix { "development" }
     "database" {
       "host" either environment = "production" {
         "prod-db.example.com"
       } {
         "localhost"
       }
       "port" 5432
       "user" "app_user"
       "password" env/get "DB_PASSWORD" |fix { "default" }
     }
     "features" {
       "dark_mode" true
       "analytics" environment = "production"
     }
   }
   ```

3. **Web Development**
   - API integrations with HTTP support
   - Data validation for web forms
   - Backend scripting

   ```clojure
   ; Simple API endpoint handler
   handle-user-registration: fn { request } {
     ; Validate the request body
     user: validate request/body {
       username: required string
       email: required email
       password: required string check { .length? >= 8 }
     }
     
     ; If validation failed, return an error
     if error? user {
       return response 400 user
     }
     
     ; Hash the password
     user .change "password" bcrypt/hash user/password
     
     ; Save to database
     db/insert "users" user
     
     ; Return success response
     response 201 { "message" "User registered successfully" }
   }
   ```

4. **Security-Critical Applications**
   - Applications requiring strict control over system resources
   - Scripts with limited permissions
   - Code that needs to be verified for authenticity

   ```clojure
   ; Running a script with security constraints
   ; $ rye --seccomp-profile=strict --landlock --landlock-profile=readonly --landlock-paths=/app/data --codesig script.rye
   
   ; The script can only read from /app/data and has limited system calls
   data: read %/app/data/sensitive.json |json/parse
   
   ; Process the data without side effects
   result: data |filter { .clearance-level <= user/clearance-level }
   
   ; Return the result (but can't write to disk or network)
   result
   ```

5. **Interactive Data Analysis**
   - REPL-based data exploration
   - Quick prototyping of data transformations
   - Creating reports and visualizations

   ```clojure
   ; Interactive data analysis in the REPL
   ; $ rye
   x> data: load\csv %sales.csv |autotype 1.0
   x> data |column? "amount" |avg |print
   ; Prints: 1245.37
   
   x> data |group-by "category" |map { [ .key .value |length? ] } |sort-by 1 'desc |take 5 |print\table
   ; Prints a table of top 5 categories by number of sales
   
   x> data |where-greater "amount" 5000 |save\csv %large-sales.csv
   ; Saves filtered data to a new CSV file
   ```

Rye combines the flexibility of dynamic languages with the safety features of more strict languages, making it particularly well-suited for scenarios where both rapid development and robustness are important.
