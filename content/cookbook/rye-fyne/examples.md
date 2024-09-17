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


![Hello fyne World](../hello_fyne_world_s.png)

```lisp
do\in fyne { 
	
	win: app .window "Hello"
	win .set-content label "Hello fyne world!"
	win .show-and-run	
}
```

## Button

What's new: `button` `v-box` `set-text` `layout-spacer` `resize`

![Button](../button.png)


```lisp
do\in fyne { 

	lab: label "I'm Waiting ..."
	btn: button "Click here" does { lab .set-text "Finally :)" }
	box: v-box [ lab layout-spacer btn ]
	
	with app .window "Button" {
		.resize size 200.0 100.0 ,
		.set-content box ,
		.show-and-run
	}
}
```

## Feedback form

What's new: `multiline` `get-text` `show-pop-up`

![Feedback form](../feedback_form.png)

```lisp
do\in fyne {

	app .window "Feedback" :win
	
	cont: v-box [
		label "Send us feedback:"
		entry .multi-line! 1 :ent
		button "Send" does {
			msg: ent .text?
			show-pop-up label "Sending: " + msg  win .canvas
		}
	]
	
	win |set-content cont |show-and-run
}
```

## Clock

What's new: `go (goroutines)`

![Date & Time](../date_time.png)

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

![My home](../my_home2.png)

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
		sleep 3 .seconds  ; sensor wake up ...
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

![Form](../form.png)


```lisp
do\in fyne {
	
	win: app .window "Form"

	btn: disable button "Sing up" does {
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
		team -> row -> col
		|set-text* o
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
