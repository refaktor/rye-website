---
title: "Live development"
date: 2024-09-13T19:30:08+10:00
draft: false
group: true
weight: 150
summary: "You can explore the wast Fyne library and combine / see and use various widgets and layout directly from Rye console (REPL)"
mygroup: true
---

*work in progress*

Fyne has a big api. It exposes multiple hundred functions / structs / properties. One benefit of Rye is that we can use Rye console itself to
explore the API. We can also start a Fyne window and then use live console (REPL) to change it, experiment with widgets in this live setting.


## Exploring Fyne binding

Keywords: `cc` `lc` `lcp` `lc\` `lcp\`

Fyne binding lives in it's own context (fyne). That's why our code generally resides in `do\in fyne` or better `do\par fyne` in our scripts.

Rye has some console specific functions that are similar to how we traverse and explore directories in general. We wouldn't normally use these in scrips,
but they are very usefull in console.

```lisp
x> lc            ; list current context, empty if we just started a console
x> lcp           ; list parent context, where all builting functions and subcontexts are
x> cc fyne       ; change context to fyne
x> lc            ; lists all fyne functions
x> lc\ "entry"   ; lists all fyne functions that match "entry"
```

&nbsp;


*`lc*` functions don't list the generic builtins that are tied to specific resources, like windows, widgets. We will add function for that too.*

*NOTE: '\' at the end of the word is a convention for "more". In general \ is used to denote variations of functions, like `load`,  `load\csv`, etc.*

## Develop from REPL

Keywords: `Rye console` `live.rye`

To put it simply, run examples/live.rye. You will get a GUI window with one label and a Rye console where you have access to the live objects defined in live.rye like 
win, app, lab. You can then construct new widgets, layouts, put widgets in layouts in console and use `win .set-content ...` to change the window.

```lisp
x> lc           ; list currently defined words
x> lab .set-text "Hello from console"
x> box: v-box { entry button "Send" does { print "test" } }
x> win .set-content box
```

