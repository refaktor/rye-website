---
title: 'HTTP Servers and websockets' 
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 500
summary: ""
---


## Http server

```rye
new-server ":8080"
|handle "/" "Hello from Rye!"
|handle "/fn" fn { r w } { write w "Hello from Rye function!" }
```

## Websockets

```rye
new-server ":8080"
|handle-ws "/echo" fn { s ctx } { forever { read s ctx :m , write s ctx m } } 
|handle-ws "/ping" fn { s ctx } { forever { reas s ctx :m = "Ping" |if { write s ctx "Pong" } } }
```
