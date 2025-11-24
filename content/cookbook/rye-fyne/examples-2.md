---
title: "Examples v2"
date: 2024-10-03T09:22:00+02:00
draft: false
group: true
weight: 55
summary: "Updated examples for the new Rye-fyne system."
mygroup: true
---

Rye-fyne has undergone a major upgrade with a new import system and updated syntax. This page contains examples updated for the new system. The new approach uses explicit Go module imports and namespaced widget/container calls for better clarity and maintainability.

If the code below doesn't always make sense to you, check out [Meet Rye](https://ryelang.org/meet_rye/), especially the part about [op and pipe-words](https://ryelang.org/meet_rye/specifics/opwords/).

**Rye-fyne examples won't work with regular Rye. You need to use [Rye-fyne](https://github.com/refaktor/rye-fyne). You can download [release binaries](https://github.com/refaktor/rye-fyne/releases/tag/v0.2.6) for Linux, Windows and Mac or [build your own](https://github.com/refaktor/rye-fyne?tab=readme-ov-file#building-from-source). The documentation on generation and build process is comming. Also tooling to generate APKs.**

## Hello fyne world

What's new: `import\go` `app/new` `widget/label` `.window` `.set-content` `.show-and-run`

We'll start with a Hello world app. The new system requires explicit imports and uses namespaced function calls for widgets and containers.

![Hello fyne World](../hello_fyne_world_s.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"

a: app/new
w: a .window "Hello"
w .set-content widget/label "Hello fyne world!"
w .show-and-run
```

## First layout

What's new: `container/hbox` `container/vbox`

Fyne uses various layout components that you can combine to declaratively lay out the widgets. The two very basic ones are now called `container/hbox` and `container/vbox`.

![layouts](../layouts.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"

a: app/new
w: a .window "Layouts"
w .set-content container/vbox [
    container/hbox [
        widget/label "Don't look RIGHT"
        widget/label "Don't look LEFT"
    ]
    widget/label "Don't look UP"
]
w .show-and-run
```

## Button and spacer

What's new: `widget/button` `.set-text` `layout/spacer` `fyne/size`

A button comes with a callback function that gets called when the button is clicked. `does` is a Rye function that creates a function with no arguments.

![Button](../button.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
layout: import\go "fyne/layout"

lab: widget/label "I'm Waiting ..."
btn: widget/button "Click here" does { lab .set-text "Finally ..." }
box: container/vbox [
    lab
    layout/spacer
    btn
]

a: app/new
w: a .window "Button"
w .resize fyne/size 200.0 100.0
w .set-content box
w .show-and-run
```

## Feedback form

What's new: `widget/entry` `widget/select` `.text?` `.multi-line!` `.place-holder!` `dialog/show-information`

Let's now combine an entry widget, make it multi-line, a select field and a button to make our first potentially practical applet.

![Feedback form](../feedback.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
dialog: import\go "fyne/dialog"

w: app/new .window "Feedback"

ent: widget/entry 
ent .multi-line! true 
sel: widget/select [ "Happy" "Normal" "Confused" ] fn { x } { } 
sel .place-holder! "How do you feel ..." 

cont: container/vbox [
    widget/label "Send us feedback:"
    ent
    sel
    widget/button "Send" does {
        msg: ent .text?
        dialog/show-information "Sending" "Sending: " ++ msg w
    }
]

w .set-content cont
w .show-and-run
```

## Live clock

What's new: `go` (goroutines) with `fyne/do`

Rye inherits awesome goroutines from Go and Fyne works very nicely with them. For GUI updates from goroutines, we use `fyne/do` to ensure thread safety.

![Date & Time](../clock.gif)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"

lab: widget/label "<date & time>"

go does {
    forever {
        fyne/do does {
            lab .set-text now .to-string
        }
        sleep 500
    }
}

w: app/new .window "Date & Time"
w .set-content lab
w .show-and-run
```

## My home

What's new: `canvas/image-from-file` `.fill-mode!`

In fact, goroutines get handy in a lot of cases. Imagine we are making an app that displays basic live information about our home. Let's throw in an example of using an image also.

This app simulates reading outside temperature from a sensor that updates every 300 milliseconds.

![My home](../home_app.gif)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
canvas: import\go "fyne/canvas"

Sensors: context {
    ; dummy sensor
    get-temperature: does { sleep 300 , 20 + random\integer 12 }
}

temp: widget/label "Reading ..."
img: canvas/image-from-file "home.png"
img .fill-mode! 2  ; original size

go does {
    sleep 3000  ; waiting for sensors to wake up
    forever {
        temp-val:: Sensors/get-temperature .to-string ++ " °C"
        fyne/do does {
            temp .set-text "Outside temp: " ++ temp-val
        }
        sleep 1000
    }
}

w: app/new .window "My Home"
w .set-content container/vbox [ img temp ]
w .show-and-run
```

## Signup form

What's new: `container/form` `widget/password-entry` `widget/check` `.disable` `.enable`

Back to more static example. Fyne has this nice concept of forms, which is very handy. You can fill the right side of the form item with any widget.

![Form](../form.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
dialog: import\go "fyne/dialog"

a: app/new
w: a .window "Form"

btn: widget/button "Sign up" does {
    dialog/show-information "Success" "You rock!" w
}
btn .disable

chk: widget/check "I fully agree" fn { v } {
    either v {
        btn .enable
    } {
        btn .disable
    }
}

cont: container/form [
    "Username" widget/entry
    "Password" widget/password-entry
    "Terms" chk
    "" btn
]

w .set-content cont
w .show-and-run
```

## 1 million items list

What's new: `widget/list` `fyne/size`

Fyne handles lists in a quite simple and effective way. To construct a list you give constructor function `widget/list` 3 functions. First returns the number of items, second constructs the widget(s) for an item and in the third you update the given widgets for a given row index.

Below we create a list with 1 million line items and it works with no performance problems.

![1 million list](../1millionlist.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"

lst: widget/list
    does { 1000000 }
    does { widget/label "num" }
    fn { i item } { item .set-text to-string i + 1 }

w: app/new .window "1 million list"
w .resize fyne/size 220.0 200.0
w .set-content lst
w .show-and-run
```

## Players' list

What's new: `widget/list` `container/hbox` `.length?` `.objects?` `fyne/size`

To me at least the main indicator of GUI framework's elegance is to see how it handles lists with composed list items. Fyne's approach does very good in this regard.

![Gastown bingo players](../list_gastown.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"

players: [ [ "WildJane" 5210 ] [ "MadBob" 4991 ] [ "GreeNoob" 12 ] ]

lst: widget/list
    does { players .length? }
    does { container/hbox [ widget/label "name" widget/label "score" ] }
    fn { i box } { 
        player: players -> i
        name-label: 0 <- box .objects?
        score-label: 1 <- box .objects?
        name-label .set-text 0 <- player
        score-label .set-text to-string 1 <- player
    }

a: app/new
w: a .window "Gastown bingo players"
w .resize fyne/size 220.0 200.0
w .set-content lst
w .show-and-run
```

## Multiplication table

What's new: `widget/table` `.row` `.col`

Fyne also has tables that are conceptually very similar to Lists.

![Multiplication table](../multiplication_table.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"

tab: widget/table
    does { [ 10 10 ] }
    does { widget/label "..." }
    fn { i o } {
        r: 1 + i .row
        c: 1 + i .col
        o .set-text r * c .to-string
    }

a: app/new
w: a .window "Multiplication table"
w .resize 330.0 400.0
w .set-content tab
w .show-and-run
```

## Percentages clock

What's new: `widget/progress-bar`

Progress bars often have to be updated from another "thread". With Fyne / Goroutines / Rye this is again no problem at all, using `fyne/do` for thread-safe updates.

![Percentages clock](../percentage_clock.gif)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"

ly: widget/label "Year:"
py: widget/progress-bar
ph: widget/progress-bar
pm: widget/progress-bar
ps: widget/progress-bar

cont: container/vbox [
    ly
    py
    widget/label "Today's hours:"
    ph
    widget/label "Minutes:"
    pm
    widget/label "Seconds:"
    ps
]

leap-year: fn { y } { 
    all [ y .mod 4 .equals 0  not y .mod 100 .equals 0  y .mod 400 .equals 0 ]
}

days-in: fn { y } { 
    either leap-year y { 366 } { 365 }
}

go does {
    forever {
        n: now
        year: n .year
        fyne/do does {
            ly .set-text "Year " ++ year .to-string
            py .set-value n .year-day / days-in year
            ph .set-value n .hour / 24.0
            pm .set-value n .minute / 60.0
            ps .set-value n .second / 60.0
        }
        sleep 500
    }
}

a: app/new
w: a .window "Percentage Clock"
w .resize 300.0 200.0
w .set-content cont
w .show-and-run
```

## Shopping list app

What's new: `widget/button-with-icon` `.objects?` `.deref` `.on-changed!` `.on-tapped!` `.on-submitted!`

This is a very simple work in progress app with simple state management. This example demonstrates more complex state management and list interactions.

![shopping list app](../shopping_list.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
theme: import\go "fyne/theme"
layout: import\go "fyne/layout"

; Simple data management
tasks: ref [
    [ false "Goat cheese" ]
    [ false "Eggs" ]
    [ false "Oats" ]
    [ false "Anchovies" ]
    [ false "Bread" ]
    [ false "A4 paper" ]
]

; Rye is mostly immutable, we are working
; on state management solution
Data: context {
    add!: fn { task } {
        print "* Adding" probe task 
    }
    remove!: fn { idx } { 
        print "* Removing" probe idx 
    }
    check!: fn { idx val } { 
        print "* Checking" probe [ idx val ]
    }
}

lst: widget/list
    does { tasks .deref .length? }
    does {
        container/hbox [
            widget/check "" fn { v } { }  ; will be updated with proper callback
            widget/label ""
            layout/spacer
            widget/button-with-icon "" theme/delete-icon fn { } { }  ; will be updated
        ]
    }
    fn { i box } {
        tasks .deref -> i :task 
        box .objects? -> 0 :chk
        box .objects? -> 1 :lbl
        box .objects? -> 3 :btn
        
        chk .set-checked 0 <- task
        lbl .set-text 1 <- task
        
        ; Update callbacks with current index
        chk .on-changed! fn { v } { Data/check! i v }
        btn .on-tapped! fn { } { Data/remove! i , lst .refresh }
    }

input: widget/entry
input .set-place-holder "Add to list here ..."
input .on-submitted! fn { x } {
    Data/add! x
    input .set-text ""
    lst .refresh
}

cont: container/border nil input nil nil [ lst ]

a: app/new
w: a .window "Shopping List"
w .resize fyne/size 300.0 300.0
w .set-content cont
w .show-and-run
```

## Tabbed settings panel

What's new: `container/app-tabs` `container/tab-item` `widget/slider` `widget/separator`

This example demonstrates a modern tabbed interface with multiple settings panels. It showcases how to organize complex interfaces using tabs, sliders for continuous values, and various interactive controls.

![Tabbed Settings](../tabbed_settings.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
theme: import\go "fyne/theme"

a: app/new
w: a .window "Settings Panel"

; General settings tab content
private {
	volume-slider: widget/slider 0.0 100.0
	volume-slider .set-value 75.0
	volume-label: widget/label "Volume: 75%"

	volume-slider .on-changed! fn { v } {
		volume-label .set-text "Volume: " ++ v ++ "%"
	}

	notifications-check: widget/check "Enable notifications" fn { v } { 
		print "Notifications:" v 
	}
	notifications-check .set-checked true

	auto-save-check: widget/check "Auto-save documents" fn { v } {
		print "Auto-save:" v
	}

	container/vbox [
		widget/label "Audio Settings"
		volume-slider
		volume-label
		widget/separator
		widget/label "Application Settings"
		notifications-check
		auto-save-check
	]
} :general-content

; About tab content
about-content: container/vbox [
    widget/icon theme/info-icon
    widget/label "Tabbed Settings Demo"
    widget/separator
    widget/label "Version 1.0.0"
]

; Create tab container
tabs: container/app-tabs [
    container/tab-item "General" general-content
    container/tab-item "About" about-content
]

w .set-content tabs
w .resize fyne/size 400.0 350.0
w .show-and-run
```

## Micro book reader

What's new: `widget/rich-text-from-markdown` `.set-min-size` `.parse-markdown`

This example shows how to create a rich text viewer with markdown support, and interactive controls. It demonstrates document-style applications.

![Micro book reader screenshot](../micro_book_reader.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"
theme: import\go "fyne/theme"

a: app/new
w: a .window "Micro book reader"

; Create rich text widget
rich-text: widget/rich-text-from-markdown "loading ..."
rich-text .wrapping! 3

page-label: widget/label "page unknown ..."

var 'page 1
next-page: closure { } { if page < 3 { change! inc page 'page } }
back-page: closure { } { if page > 1 { change! decr page 'page } }
load-page: closure { } {
	rich-text .parse-markdown Read to-file join [ "page" page ".md" ]
	page-label .set-text join [ "Page: " page ]
}

load-page

; Create toolbar
toolbar: container/hbox [
    widget/button-with-icon "Previous" theme/navigate-back-icon does {
		back-page
		load-page
    }
    widget/button-with-icon "Next" theme/navigate-next-icon does {
		next-page
		load-page
    }
]

; Main layout using border container
main-content: container/border 
  toolbar          ; top
  container/hbox [ ; bottom
	page-label
  ]
  nil              ; left
  nil              ; right
  [ rich-text ]    ; center (as list for multiple items)

w .set-content main-content
w .resize fyne/size 500.0 500.0
w .show-and-run
```

## Get my IP

What's new: `https // Get` `.clipboard` `.set-content` `fix`

This one, I actually use daily. HTTP call happens in a goroutine so it doesn't block UI rendering and it refreshes the UI using fyne/do to prevent race conditions.

Html parsing is somewhat patched together, but "it works", and it a simple demo of failure handling.

![Get my IP screenshot](../get_my_ip.png)

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"

w: app/new .window "Get my IP"

url: https://ifconfig.me
lab: widget/label "retrieving ..."
btn: widget/button "Copy" fn { } {
    w .clipboard .set-content lab .text?
}

get-ip: does {
   Get url
   |^fix { embed url "couldn't load {{}}" }
   |html->markdown
   |^fix { "couldn't parse html" }
   |Submatch?* regexp "IP Address \*\*([0-9.]+)\*\*"
   |^fix { "couldn't find the IP pattern" }
}

go does {
    forever {
        fyne/do does {
            lab .set-text get-ip
        }
        sleep 1 .minutes
    }
}

w .set-content container/hbox [ lab btn ]
w .show-and-run
```

## My Movie Database (SQLite)

What's new: `Open` `Query` `Exec` `on-change` 


![MY Movie Database](../mymoviedb.png)

[See the video on YouTube](https://www.youtube.com/watch?v=Y1JHHS2iq2Q*).

```lisp
fyne: import\go "fyne"
app: import\go "fyne/app"
widget: import\go "fyne/widget"
container: import\go "fyne/container"

; State
Movies: context {
    cl: ?closure
    var 'movies { } ; |on-change { update-entries }
    var 'index 0    ; |on-change { update-entries }
    db: Open sqlite://movies.db

    current?: cl { } { 
        movies -> index 
        |fix { dict { name: "" score: 0 } } }
    current-id?: cl { } { current? -> 0 }
    clamp-index!: cl { i } {
        clamp 0 movies .max-idx? 
        |modify! 'index }
    next!: cl { } { clamp-index! inc index }
    prev!: cl { } { clamp-index! decr index }
    new-index!: cl { } { movies .length? |modify! 'index }
    is-new: cl { } { index > ( max-idx? movies ) }

    insert!: cl { name score } {
        db .Exec { insert into movies ( name , score )
                   values ( ?name , ?score ) } }
    update!: cl { id name score } {
        db .Exec { update movies set name = ?name ,
                   score = ?score where id = ?id } }
    delete!: cl { id } {
        db .Exec { delete from movies where id = ?id } }
    reload!: cl { } {
        db .Query { select id , name , score
                    from movies order by id }
        |modify! 'movies
        clamp-index! index } }

; GUI
a: app/new
w: a .window "My Movie Database [MYMDb]"
w .resize fyne/size 300.0 250.0

name-entry: widget/entry
score-entry: widget/entry

update-entries: does {
    name-entry .set-text "name" <- Movies/current?
    score-entry .set-text to-string "score" <- Movies/current? }

do\inside Movies { ; seems cleaner this way
    'movies .on-change { update-entries } 
    'index  .on-change { update-entries } }

prev-btn: widget/button "Previous" does { Movies/prev! }
next-btn: widget/button "Next" does { Movies/next! }
new-btn: widget/button "New" does { Movies/new-index! }

save-btn: widget/button "Save" does {
    name: name-entry .text? 
    score: score-entry .text? .to-integer
    do\in Movies {
        either is-new { insert! name score } 
                      { update! current-id? name score }
        reload! } }

delete-btn: widget/button "Delete" does {
    do\in Movies { delete! current-id? , reload! } }

w .set-content container/vbox [
    container/hbox [ prev-btn next-btn ]
    widget/label "Name:"
    name-entry
    widget/label "Score:"
    score-entry
    container/hbox [ new-btn save-btn delete-btn ] ]

Movies/reload!

w .show-and-run
```


## Changes from v1

These examples demonstrate the new Rye-fyne syntax with explicit imports and namespaced function calls. 

The new system is somewhat more verbose, but provides better clarity about which Go modules are being used and makes the code more maintainable and contexts explorable in Rye console.

You can still find the [old examples](../examples) for comparison.

Key changes in Fyne:
- **Explicit imports**: `import\go` statements for each Fyne module
- **Namespaced calls**: `widget/label` instead of `label`, `container/vbox` instead of `v-box`
- **Clearer structure**: Better separation between Fyne components
- **Thread safety**: `fyne/do` for GUI updates from goroutines

Key changes in Rye:
- **Const by default**: `set-word:` produces a constant
- **Explicit variables**: `var 'word value` produces a variable
- **Observable variables**: `'word .on-change { code }` defines code to evaluate when a variable changes


Created: **10/26/2025**

## Related links

* [Examples as files](https://github.com/refaktor/rye-fyne/tree/main/examples) - these examples as Rye files that you can run
* [Rye-Fyne project](https://github.com/refaktor/rye-fyne) - Rye extended with Fyne, code, examples, binaries
* [Rye language](https://github.com/refaktor/rye) - main language repository
* [Fyne GUI library](https://fyne.io) - Fyne GUI library
* [ryegen](https://github.com/refaktor/rye-fyne) - binding generator
