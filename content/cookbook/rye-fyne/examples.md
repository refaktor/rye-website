---
title: "Examples"
date: 2024-08-13T19:30:08+10:00
draft: false
group: true
weight: 50
summary: "An growing list of simple GUI examples."
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

## Hello fyne world

What's new: `app` `window` `label` `set-content` `show-and-run`

We'll start this with just a Hello world app. We see the basic building blocks (functions / constructors) of every Fyne app here, an `app`, a `window` and the first widget `label` and two functions `set-content` and `show-and-run`.

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

Fyne uses various layout components that you can combine to declaratively lay out the widgets. The two basic ones age horizontal and vertical boxes `h-box` and `v-box`.

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

A button comes with a callback function that gets called when the button is clicked. `does` is a Rye function that creates function with no arguments. 

Since we want a button in this case to be at the bottom of our window, when we resize it, we use a `layout-spacer` which takes that extra space that is available, in this case vertically. 

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

What's new: `multi-line!` `get-text` `show-pop-up`

Let's now combine these, change entry to multi-line and add a select field to make our first potentially practical applet.

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

*by Rye convention a `noun!` means set-noun, like `noun?` means get-noun. This is used in Fyne bindings to name functions that
set and get Fynes structs' properties.*

## Live clock

What's new: `go (goroutines)`

Rye inherits the awesome goroutines from Go and Fyne seems to works very nicely with goroutines. So doing live updates to the GUI (main thread) from a paralel "process", like a goroutine, is just as simple as it could be.

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

Back to more static world. Fyne has this nice concept of `form`, which is very handy. You can fill the right side of the form with any widget.

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

## Gastown bingo players

What's new: `list` `h-box` `set-title`

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

![shoping list app](../shopping_list.png)

```lisp
Data: context {

	add!: fn { task } { change! tasks .add-row task 'tasks }
	remove!: fn { idx } { print "Delete row: " + idx } ; tasks .remove-row! idx }
	check!: fn { idx val } { print "Check row: " + idx } ; tasks .update-row idx [ 'done val ] } }
	
	tasks: spreadsheet { 'done 'text } {
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
	does { length? Data/tasks }
	does {
		idx: hide label "idx"
		h-box [
			check "" 
			  fn\par { v } current { print "Check row: " + idx .text? }
			label ""
			idx
			layout-spacer
			button-with-icon "" delete-icon 
			  fn\par { } current { Data/remove! to-integer idx .text? }
		]
	}
	fn { i box } { 
		set! box .objects? { chk lbl hdn xo btn }
		chk .set-checked 0 <- i <- Data/tasks
		lbl .set-text 1 <- i <- Data/tasks
		hdn .set-text to-string i
		; btn .on-tapped! 
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
