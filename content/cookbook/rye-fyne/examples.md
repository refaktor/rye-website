---
title: "Examples"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 50
summary: "An growing list of simple GUI examples."
mygroup: true
---

Rye started out focused on 'backed development' and 'interactive use in console' and it still is. But **[Go's Fyne GUI library](https://fyne.io/)** was just too tempting not to try to integrate. It's easy to deploy, works on desktop and mobile platforms and keeps getting better.

If the code below doesn't always make sense to you, check out [Meet Rye](https://ryelang.org/meet_rye/), especially the part about [op and pipe-words](https://ryelang.org/meet_rye/specifics/opwords/).

<!-- Rye started out focused on 'backed development' and 'interactive use in console'. However, with the continuous improvement of Go's Fyne GUI framework, we were tempted to explore integrating it to see how Rye could handle GUI code.

We were integrating Fyne into Rye manually, but at some point Darwin came around and proposed to create a tool to automatically generate the bindings. It's still all work in progress, but the results are here. -->

<!-- ## No GUI dialect

If you’re familiar with REBOL, the language Rye is based on, you might recall its famous GUI dialect. Rye, somewhat controversially, doesn’t have a dedicated GUI dialect. Instead, we rely on Rye's core language, which we believe is flexible enough to elegantly create  GUI structures. -->

## Hello fyne world

What's new: `app` `window` `label` `set-content` `show-and-run`

We'll start with a Hello world app. We can see the basic building blocks of every Fyne app here, constructor functions for `app`, `window` and a widget `label` and functions `set-content` and `show-and-run`.

![Hello fyne World](../hello_fyne_world_s.png)

```lisp
do\in fyne { 
	
	win: app .window "Hello"
	win .set-content label "Hello fyne world!"
	win .show-and-run	
}
```

## First layout

What's new: `h-box` `v-box`

Fyne uses various layout components that you can combine to declaratively lay out the widgets. The two very basic ones are horizontal and vertical boxes `h-box` and `v-box`.

![layouts](../layouts.png)

```lisp
do\in fyne { 
	
	win: app .window "Layouts"
	win .set-content v-box [ 
		h-box [
			label "Don't look RIGHT"
			label "Don't look LEFT"
		]
		label "Don't look UP"
	]
	win .show-and-run	
}
```


## Button and spacer

What's new: `button` `set-text` `layout-spacer` `resize`

A button comes with a callback function that gets called when the button is clicked. `does` is a Rye function that creates a function with no arguments. 

Since we want the button in this case to remain at the bottom of our window, when we resize it, we also add a `layout-spacer`, which takes that extra space that is available, in this case vertically. 

![Button](../button.png)

```lisp
do\in fyne { 

	lab: label "I'm Waiting ..."
	btn: button "Click here" does { lab .set-text "Finally ..." }
	box: v-box [ lab layout-spacer btn ]
	
	with app .window "Button" {
		.resize size 200.0 100.0 ,
		.set-content box ,
		.show-and-run
	}
}
```

## Feedback form

What's new: `entry` `multi-line!` `select` `get-text` `show-pop-up`

Let's now combine an entry widget, make it multi-line, a select field and a button to make our first potentially practical applet.

![Feedback form](../feedback.png)

```lisp
do\in fyne {

	app .window "Feedback" :win
	
	cont: v-box [
		label "Send us feedback:"
		entry .multi-line! true :ent
		select [ "Happy" "Normal" "Confused" ] fn { v } { }
		|place-holder! "How do you feel ..."
		button "Send" does {
			msg: ent .text?
			show-pop-up label "Sending: " + msg  win .canvas
		}
	]
	
	win |set-content cont |show-and-run
}
```

&nbsp;

*By Rye convention a `noun!` means set-noun, and `noun?` means get-noun. This is also used in Fyne bindings to name functions that set and get Fynes structs' properties.*

## Live clock

What's new: `go (goroutines)`

Rye inherits awesome goroutines from Go and Fyne seems to works very nicely with them. So doing live updates to the GUI (main thread) from a paralel "process", like a goroutine, is just as simple as it could be.

![Date & Time](../clock.gif)

```lisp
do\in fyne { 

	lab: label "<date & time>"

	update-time: does { lab .set-text now .to-string }

	go does { forever { sleep 100 , update-time } }
	
	with app .window "Date & Time" {
		.set-content lab ,
		.show-and-run
	}
}
```

## My home

What's new: `canvas-image-from-file` `fill-mode!`

In fact, goroutines get handy in a lot of cases. Imagine we are making an app that display basic live information about our home. Let's throw in an example of using an image also. 

This app simulates reading outside temperature from a sensor that updates every 300 milliseconds.

![My home](../home_app.gif)

```lisp
Sensors: context {
	; dummy sensor
	get-temperature: does { sleep 300 , 20 + random-integer 12 }
}

do\par fyne {

	win: app .window "My Home"

	temp: label "Reading ..."
	img: canvas-image-from-file "home.png"
	img .fill-mode! canvas-image-fill-original
	
	go fn { } {
		sleep 3 .seconds  ; waiting for sensors to wake up
		forever {
			Sensors/get-temperature .to-string + " °C"
			|concat* "Outside temp: "
			|set-text* temp ,
			sleep 1 .seconds
		}
	}
	
	with win  {
		.set-content v-box [ img temp ] ,
		.show-and-run
	}
}
```


## Signup form

What's new: `form` `password-entry` `check` `disable` `enable`

Back to more static world. Fyne has this nice concept of `form`-s, which is very handy. You can fill the right side of the form item with any widget.

![Form](../form.png)


```lisp
do\in fyne {
	
	win: app .window "Form"

	btn: disable button "Sign up" does {
		show-pop-up label "You rock!"  win .canvas
	} 

	cont: form [
		form-item "Username" entry
		form-item "Password" password-entry 
		form-item "Terms" chk: check "I fully agree" 
		                       fn { v } { btn .enable }
		form-item "" btn
	]

	win .set-content cont |show-and-run
}
```

## 1 million items list

What's new: `list`

Fyne handles lists in a quite simple and effective way. To construct a list you give constructor function `list` 3 functions. 
First returns the number of items, second constructs and the widget(s) for an item and in the third you update the given widgets
for a given row index. In this way Fyne reuses list item widgets and can basically display lists of any size.

Below we create a list with 1 million line items and it works with no performance problems.

![1 million list](../1millionlist.png)

```lisp
do\par fyne {

	lst: list does { 1000000 }
	does { label "num" }
	fn { i item } { item .set-text to-string i + 1 }

	app .window "1 million list"
	|resize size 220.0 200.0
	|set-content lst
	|show-and-run
}
```

## Players' list

What's new: `list` `h-box` `set-title`

To me at least the main indicator of GUI framework's elegance is to see how it handles lists with composed list items. Fyne's approach does very good in this regard.

![Gastown bingo players](../list_gastown.png)

```lisp
players: { { "WildJane" 5210 } { "MadBob" 4991 } { "GreeNoob" 12 } }

do\par fyne { 

	win: app .window "Wroom ..."
	
	lst: list does { length? players }
	does { h-box [ label "name" label "score" ] }
	fn { i box } { set! box .objects? { name score }
		name .set-text 0 <- i <- players
		score .set-text to-string 1 <- i <- players }
	
	with win {
		.resize size 220.0 200.0 ,
		.set-content lst ,
		.set-title "Gastown bingo players" ,
		.show-and-run
	}
}
```

## Multiplication table

What's new: `table` `row?` `col?`

Fyne also has tables that are conceptually very similar to Lists.

![Multiplication table](../multiplication_table.png)

```lisp
do\par fyne { 

	win: app .window "Multiplication table"
	
	tab: table does { [ 10 10 ] }
	does { label "..." }
	fn { i o } {
		r: 1 + i .row? , c: 1 + i .col?
		o .set-text to-string r * c
	}
	
	with win {
		.resize size 330.0 400.0 ,
		.set-content tab ,
		.show-and-run
	}
}
```

## Percentages clock

What's new: `progress-bar`

Progress bars often have to be updated from another "thread". With Fyne / Goroutines / Rye this is again no problem at all.

![Percentages clock](../percentage_clock.gif)

```lisp
do\in fyne {

	app .window "Percentage Clock" :win
	
	cont: v-box [
		label "Year:" :ly
		progress-bar :py
		label "Today's hours:"
		progress-bar :ph
		label "Minutes:"
		progress-bar :pm
		label "Seconds:"
		progress-bar :ps
	]

	leap-year: fn { y } { all { y .multiple-of 4  not y .multiple-of 100  not y .multiple-of 400 } }
	days-in: fn { y } { .leap-year .either { 366 } { 365 } }
	
	go fn\par { } current {
		forever {
			now
			|pass { .year? ::y |concat* "Year " |set-text* ly }
			|pass { .year-day? / days-in y  |set-value* py }
			|pass { .hour?     / 24 |set-value* ph }
			|pass { .minute?   / 60 |set-value* pm }
			|pass { .second?   / 60 |set-value* ps }
			sleep 500
		}
	}
	
	win |resize size 300.0 200.0 |set-content cont |show-and-run
}
```

## CSV file in a table

What's new: `header?` `show-header-row!` `update-header!`

Rye comes with the Spreadsheet value type. Here we load it from CSV and then display it in a Fyne table.

![LOTR team](../lotr_team.png)

```lisp
team: load\csv %lotr_team.csv

do\par fyne { 

	win: app .window "LOTR Team"
	
	tab: table does { [ team .length? team .header? .length? ] }
	does { label "......................." }
	fn { i o } {
		row: i .row? , col: i .col?
		team -> row -> col |set-text* o
	}
	|show-header-row! true
	|update-header! fn { i o } {
		team .header? -> i .col? |to-string .set-text* o
	}
	
	with win {
		.resize size 350.0 300.0 ,
		.set-content tab ,
		.show-and-run
	}
}
```


## Shopping list app

What's new: `button-with-icon` `objects?` `set-checked` `place-holder!` `on-submitted!` `refresh`

This is a very simple work in progress app with simple state management using a Rye's spreadsheet value type. Spreadsheets were like other values in Rye basically immutable data strucures, so they can't be
efficietly used as a app-level "database". We are changing some things so they can also be usefull in this situation. While the default use is still immutable.

![shoping list app](../shopping_list.png)

```lisp
rye .needs { fyne }

Data: context {

	add!: fn { task } { tasks .add-rows! task }
	remove!: fn { idx } { tasks .remove-row! idx + 1 }
	check!: fn { idx val } { tasks .update-row! idx + 1 dict [ "done" val ] }
	
	tasks: ref spreadsheet { "done" "text" } {
		0 "Goat cheese"
		0 "Eggs"
		0 "Oats"
		0 "Anchovies"
		0 "Bread"
		0 "A4 paper" }
}

do\par fyne { 

	win: app .window "Shopping List"

	lst: list
	does { length? deref Data/tasks }
	does {
		idx: hide label "idx"
		h-box [
			check "" fn\par { v } current
			{ Data/check! to-integer idx .text? v , lst .refresh }
			label ""
			idx
			layout-spacer
			button-with-icon "" delete-icon fn\par { } current
			{ Data/remove! to-integer idx .text? , lst .refresh }
		]
	}
	fn { i box } { 
		set! box .objects? { chk lbl hdn xo btn }
		chk .set-checked 0 <- i <- deref Data/tasks
		lbl .set-text 1 <- i <- deref Data/tasks
		hdn .set-text to-string i
	}

	input: entry
	|place-holder! "Add to list here ..."
	|on-submitted! fn { x } {
		Data/add! [ 0 x ]
		input .set-text ""
		lst .refresh
	}

	cont: border nil input nil nil [ lst ]
	
	with win {
		.resize size 300.0 300.0 ,
		.set-content cont ,
		.show-and-run
	}
}
```

## There is more

These are not all Fyne's components at all, so this document will get part 2 in the future.  Thanks for reading this so far.

Current version: **09/24/2024**

## Related links

* [Rye language](https://ryelang.org)
* [Fyne GUI library](https://fyne.io)

* [Rye language](https://github.com/refaktor/rye) - main language repository
* [Rye-Fyne project](https://github.com/refaktor/rye-fyne) - Rye extended with Fyne
* [ryegen](https://github.com/refaktor/rye-fyne) - binding generator
