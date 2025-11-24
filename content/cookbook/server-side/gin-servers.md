---
title: "Gin HTTP Servers"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 55
summary: "Simple examples using the Gin web framework in Rye."
mygroup: true
---

<style>
  pre > code {
      __width: 440px;
      border-radius: 5px;
      __padding: 16px;
      __background-color: #3e3e3e;
      font-size: 13px;
  }
</style>

> This page is still heavily work in progress!

## Hello world with Gin

The simplest Gin server example demonstrating basic routing and responses.

What's new: `router` `GET` `String` `Run`

```lisp
rye .needs { gin }

do\in gin {
    srv: router
    
    srv .GET "/hello" .{
        .String status\OK "Hello, Gin World!"
    }
    
    Run srv ":8080"
}
```

The `router` creates a new Gin engine with default middleware (logging and recovery). The `GET` method adds a route handler, and `String` sends a text response.

## URL parameters and query strings

Extract data from URLs using path parameters and query parameters.

What's new: `Param` `Query` `JSON`

```lisp
rye .needs { gin }

do\in gin {
    srv: router
    
    ; Path parameters
    srv .GET "/users/:id" .{
        .Param "id" :user-id ,
        .String status\OK "User ID: " ++ user-id
    }
    
    ; Query parameters with JSON response
    srv .GET "/search" .{
        .Query "q" :query ,
        .Query "limit" :limit ,
        result: dict [
            "query" query
            "limit" limit
            "found" 42
        ] ,
        .JSON status\OK result
    }
    
    Run srv ":8080"
}

; Try: /users/123 or /search?q=rye&limit=10
```

The `Param` function extracts URL path parameters (`:id`), while `Query` gets query string parameters. `JSON` automatically converts dictionaries to JSON responses.

## Form handling and POST requests

Handle form submissions and JSON payloads from POST requests.

What's new: `POST` `PostForm` `ShouldBindJSON`

```lisp
rye .needs { gin }

do\par gin {
    srv: router
    
    ; Form data handling
    srv .POST "/contact" .{
        .PostForm "name" :name ,
        .PostForm "email" :email ,
        .PostForm "message" :msg ,
        response: "Thank you " ++ name ++ "! Message: " ++ msg ,
        .String status\OK response
    }
    
    ; JSON payload handling  
    srv .POST "/api/users" .{
        data: .ShouldBindJSON ,
        response: dict {
            "received" data
            "id" 123
            "status" "created"
        } ,
        .JSON status\created response
    }
    
    Run srv ":8080"
}
```

The `PostForm` function extracts form field values, while `ShouldBindJSON` automatically parses JSON request bodies into Rye dictionaries.

## Route groups and middleware

Organize routes with prefixes and apply middleware for cross-cutting concerns.

What's new: `Group` `Use` `Next` `Header`

```lisp
rye .needs { gin }

do\par gin {
    srv: router
    
    ; Simple logging middleware
    srv .Use .{
        print [ "Request from:" .ClientIP ] ,
        .Next
    }
    
    ; API route group
    api: srv .Group "/api/v1"
    
    api .GET "/status" .{
        .JSON status\OK dict { "status" "ok" "version" "1.0" }
    }
    
    api .GET "/time" .{
        .Header "Content-Type" "application/json" ,
        .JSON status\OK dict [ "timestamp" now .to-string ]
    }
    
    Run srv ":8080"
}

; Try: /api/v1/status or /api/v1/time
```

The `Group` function creates route groups with common prefixes. `Use` adds middleware that runs before route handlers, and `Next` continues to the next handler in the chain.

## Simple authentication

Basic authentication middleware and session management.

What's new: `GetHeader` `AbortWithStatus` `session\login` `session\logout`

```lisp
rye .needs { gin }

do\par gin {
    srv: router
    
    ; Auth middleware for protected routes
    auth-required: .{
        auth: .GetHeader "Authorization" ,
        either equals? auth "Bearer secret-token" {
            .Next
        } {
            .AbortWithStatus status\unauthorized
        }
    }
    
    ; Public login endpoint
    srv .POST "/login" .{
        .PostForm "username" :username ,
        .PostForm "password" :password ,
        either equals? password "demo123" {
            session\login . username username ,
            .JSON status\OK dict { "message" "Login successful" }
        } {
            .JSON status\unauthorized dict { "error" "Invalid credentials" }
        }
    }
    
    ; Protected routes
    protected: srv .Group "/protected"
    protected .Use ?auth-required
    
    protected .GET "/profile" .{
        user: session\get-user . ,
        .JSON status\OK user
    }
    
    protected .POST "/logout" .{
        session\logout . ,
        .JSON status\OK dict { "message" "Logged out" }
    }
    
    Run srv ":8080"
}
```

This example shows middleware-based authentication using headers and simple session management functions for user login/logout state.

## HTML templates

Serve dynamic HTML using templates with data.

What's new: `LoadHTMLGlob` `HTML`

```lisp
rye .needs { gin }

do\par gin {
    srv: router
    
    ; Load HTML templates
    srv .LoadHTMLGlob "templates/*"
    
    srv .GET "/" .{
        data: dict [
            "title" "Rye + Gin"
            "message" "Welcome to our web app!"
            "items" list { "Item 1" "Item 2" "Item 3" }
        ] ,
        .HTML status\OK "index.html" data
    }
    
    srv .GET "/user/:name" .{
        .Param "name" :username ,
        data: dict [
            "title" "User Profile" 
            "username" username
            "joined" "2024"
        ] ,
        .HTML status\OK "profile.html" data
    }
    
    Run srv ":8080"
}
```

The `LoadHTMLGlob` function loads HTML templates from a directory pattern, and `HTML` renders templates with data passed as Rye dictionaries.

## REST API example

A complete REST API demonstrating all HTTP methods and status codes.

What's new: `PUT` `DELETE` status codes

```lisp
rye .needs { gin }

do\par gin {
    srv: router
    
    ; In-memory storage
    users: list [ 
        dict [ "id" 1 "name" "Alice" "email" "alice@example.com" ]
        dict [ "id" 2 "name" "Bob" "email" "bob@example.com" ]
    ]
    
    api: srv .Group "/api/users"
    
    ; GET /api/users - list all users
    api .GET "" .{
        .JSON status\OK dict [ "users" users ]
    }
    
    ; GET /api/users/:id - get user by ID
    api .GET "/:id" .{
        .Param "id" :id ,
        user: find users .{ equals? get . "id" to-integer id } ,
        either user { 
            .JSON status\OK user
        } {
            .JSON status\not-found dict { "error" "User not found" }
        }
    }
    
    ; POST /api/users - create new user
    api .POST "" .{
        data: .ShouldBindJSON ,
        new-user: data .set "id" length users + 1 ,
        users: append users new-user ,
        .JSON status\created new-user
    }
    
    ; PUT /api/users/:id - update user
    api .PUT "/:id" .{
        .Param "id" :id ,
        data: .ShouldBindJSON ,
        updated: data .set "id" to-integer id ,
        .JSON status\OK updated
    }
    
    ; DELETE /api/users/:id - delete user
    api .DELETE "/:id" .{
        .Param "id" :id ,
        .JSON status\OK dict [ 
            "message" "User " ++ id ++ " deleted" 
        ]
    }
    
    Run srv ":8080"
}
```

This comprehensive example shows a complete REST API with all HTTP methods and appropriate status codes for different operations.
