---
title: "Tables 1/3"
weight: 50
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Functions for working with table value type."
mygroup: true
toc: true
---

&nbsp;


Most humans are visual thinkers. We think about and manipulate information in our mind, much like we organize and handle objects in our physical world. We deal with single items, we create lists of items, and, one level up, we arrange items 
in two-dimensional grids. 

We were jotting down information in tabular format back in Babylon.

![babylon tablet](../babylon.webp)


When working with information on a computer, the principle remains the same. We handle single units of information, lists of units, and, one level up, two-dimensional grids of units. The epitome of this organization are the table apps.

So why should **high level** programming languages lack this format. <!-- Why should we have to abandon this and adjust to computer constructs and "speak". -->

You can not have a higher level (closer to human) language, without also higher level (closer to human) value types. These then enable higher level operations on information you are working with.

<!-- To create higher-level programming language, we need higher-level value types. These higher-level types then in turn enable higher level (more declarative, in computer terms) operations on our information making whore ordeal better. -->


## Constructors

You can construct a by calling a function `table` and providing a block of column names and a block of values.

```lisp
table { 'title 'year } { "1984" 1949 "I, Robot" 1950 "Dune" 1965 }
; creates a table with two columns and 3 rows
;
; | title    | year |
; +-----------------+
; | 1984     | 1949 |
; | I, Robot | 1950 |
; | Dune     | 1965 |
```
&nbsp;

There are two variants of the function that allow the data part to be organized in blocks of rows or blocks of columns.

```lisp
table\rows { "title" "year" }
 { { "Foundation" 1951 } { "Neuromancer" 1984 } { "Snow crash" 1992 } }
; creates a table with two columns and 3 rows
; 
; | title      | year |
; +-------------------+
; | Foundation | 1951 |
; | Neuromancer| 1984 |
; | Snow crash | 1992 |

table\columns { 'title 'year }
 { { "Foundation" "Neuromancer" "Snow crash" } { 1951 1984 1992 } }
; creates the same looking table as above
```

<!--> **variant** in Rye is a separate function with similar goal that usually accepts a little different arguments. -->

&nbsp;

## Loading a table

Besides creating a table in code, you can also get a table directly from operations like loading a CSV file, XLSX file or querying a database.

### From CSV file

Loading a CSV file results in a table.

```lisp
$ cat > battle_of_darkness.csv
ship,population
Natural selection,15000
Blue space,2000
Ultimate law,5000

$ rye

load\csv battle_of_darkness.csv
; returns a table with 2 columns and 3 rows
; 
; | ship              | population |
; +--------------------------------+
; | Natural selection | 15000      |
; | Blue space        | 2000       |
; | Ultimate law      | 5000       |
```
&nbsp;

### Autotype

CSV files don't have type information. So values are loaded into the table as strings. You can use Rye functions to convert 
values to the types you need. Or you can use function `autotype`. 

Autotype samples the table columns and converts values
based on their format.

If we wanted to **sum** the population in the previous example, we could convert strings to integers ourselves. 

```lisp
load\csv battle_of_darkness.csv
|column? 'population 
|map { .to-integer } 
|sum
; returns 22000
```
&nbsp;

> **column?** returns a single column as a block. '?' at the end is a convention for 'get-' in Rye.

Or we could use the `autotype` function which will try to figure out value types and convert to them.

```lisp
load\csv battle_of_darkness.csv 
|autotype 0.95
|column? 'population 
|sum
; returns 22000
```
&nbsp;

The decimal argument determines what level of consistency must a column have to be accepted.


### From SQL query

Querying a database also returns the table value type.

```lisp
$ cat | sqlite3 theseus.db
create table crew (id integer, name text, race text);
insert into crew values 
 (1, "Siri Keeton", "human"),
 (2, "Jukka Sarasti", "vampire"),
 (3, "Susan James", "human");

$ rye

open sqlite://theseus.db
|query { select * from crew }
; returns a table with 3 columns and 3 rows
; 
; | id | name            | race      |
; +----------------------------------+
; | 1  | Siri Keeton     | human     |
; | 2  | Jukka Sarasti   | vampire   |
; | 3  | Susan James     | human     |
```
&nbsp;

### From Excel file

Rye can also load .xlsx files directly.

```lisp
$ wget ryelang.org/cookbook/travelog.xlsx

$ rye

open\xlsx %travelog.xlsx
; returns a table with 3 columns and 5 rows
; 
; | Name            | Area  | Comment        |
; +------------------------------------------+
; | Chiba City      | Japan | somewhat damp  |
; | Zion Cluster    | Space | smells of pot  |
; | Freeside resort | Orbit | couldn't relax |
```

&nbsp;

&nbsp;

![Cyberpunk docks](../cyberpunk_docks_1.webp)


## Creating a table in code

Creating tables in code is not really hard.

### Prime number multiples

Here we create a table of multiples of the first five prime numbers: 

```lisp
primes: { 2 3 5 7 11 }

map primes { ::x map primes { * x } }
|table\rows* map primes ?to-string
|display

; displays:
;
; | 2   | 3   | 5   | 7   | 11   |
; +------------------------------+
; | 4   | 6   | 10  | 14  | 22   |
; | 6   | 9   | 15  | 21  | 33   |
; | 10  | 15  | 25  | 35  | 55   |
; | 14  | 21  | 35  | 49  | 77   |
; | 22  | 33  | 55  | 77  | 121  |
```
&nbsp;

> **double colon** denotes a mod-word, set word can only set value once in given context. They are meant to be a little hard to look at.

> **star** at the end of a word makes a function take second argument vs. the first from the left.

### Randomized data

Here we generate a table of 3 random Fremen with random water reserves from 0 to 5 liters.

```lisp
names: { "Harah" "Jamis" "Chani" "Stilgar" "Farok" }

produce 3 { } { + [ random names  random\integer 5 ] }
|table* { 'name 'water } 
|display

; displays:
;
; | name    | water |
; +---------.-------+
; | Chani   | 2     |
; | Farok   | 1     |
; | Stilgar | 4     |
```
&nbsp;

> **produce** *is a HOF-like function that accepts a number of repetitions, an initial value and a block of code that is applied to
>  the block's last resulting value.*

&nbsp;

## Inspecting a table

When we load a table, we usually want to first see its shape, its column names, maybe types, and most of all see it, or see a sample of it.

### Download and load

Although it's hard to beat `wget`, we'll use Rye itself to download a file this time.

```lisp
open https://ryelang.org/cookbook/ + file: %adversaries.xlsx
|copy open\write file

spr: load\xlsx file
; returns a table
; 
; | Year | Name               | Species              | Keywords                                |
; +--------------------------------------------------------------------------------------------+
; | 10191| Vladimir Harkonnen | Human                | Scheming, Ruthless, Manipulative        |
; | 1984 | Big Brother        | Human (Entity)       | Oppressive, Omnipresent, Tyrannical     |
; | 2154 | RDA Corporation    | Human (Organization) | Corporation, Extractive                 |
; | 2371 | The Borg           | Cyborg Collective    | Assimilation, Unstoppable, Hive-Mind    |
; | 3021 | The Mule           | Mutant               | Charismatic, Unpredictable, Destructive |
; | 2029 | Skynet             | AI                   | Cold, Calculating, Ubiquitous           |
; ...
```
&nbsp;

### Size and shape

Rye console showed use the number of columns and rows, but we can do it in code also. 

```lisp
spr .probe
; prints: 
; [Table (4 24)]

; get the number of rows (a general length? function)
spr .length? 
; returns 24

; get the number of columns (general length? on a header)
spr .header? .length?
; returns 5
```
&nbsp;

### Column names and types

Table header names tell us what kind of information our table is holding.

```lisp
; to see the column names, the haders of the spredsheet
spr .header? 
; returns column names
; { "Year" "Name" "Species" "Keywords" }

; get the types of columns
spr .types?
; returns a block of words
; { integer text text text text }
```
&nbsp;

Note that columns by itself don't enforce uniform types. Function `types?` makes a small sample of rows and if types match returns the types found.

In future you will be able to use Rye's validation dialect, if you will want to enforce types or even more detailed conditions.

### Displaying a table

But nothing beats just displaying the table. 

Many times tables are too large for casual viewing. In these cases we could use a table function `head`, to
display just the first N rows.

```lisp 
; display whole table
spr .display

spr .head 5 |display
; Displays:
;
; | Year   | Name               | Species              | Keywords                                |
; +----------------------------------------------------------------------------------------------+
; | 10191  | Vladimir Harkonnen | Human                | Scheming, Ruthless, Manipulative        |
; | 1984   | Big Brother        | Human (Entity)       | Oppressive, Omnipresent, Tyrannical     |
; | 2154   | RDA Corporatio     | Human (Organization) | Corporation, Extractive                 |
; | 2371   | The Borg           | Cyborg Collective    | Assimilation, Unstoppable, Hive-Mind    |
; | 3021   | The Mule           | Mutant               | Charismatic, Unpredictable, Destructive |
```

&nbsp;


### Sampling

`head` above helps us display first 5 rows, but we can use other general Rye's functions to get a slice of rows, a sample or last 
N rows.

These are general Rye functions that also work on other collection value types, like blocks or strings.


```lisp
; we could get 5 rows from tenth on
spr .slice 10 5 |display

; or we can get a random sample
spr .sample 5 |display
; Displays:
;
; | Year   | Name                     | Species              | Keywords                                |
; +----------------------------------------------------------------------------------------------------+
; | 2154   | RDA Corporation          | Human (Organization) | Corporation, Extractive                 |
; | 3021   | The Mule                 | Mutant               | Charismatic, Unpredictable, Destructive |
; | 2054   | Precrime                 | Human (System)       | Authoritarian, Flawed, Predictive       |
; | 2023   | VIKI                     | AI                   | Logical, Overbearing, Ruthless          |
; | 2015   | Ultron                   | AI                   | Ambitious, Intelligent, Destructive     |

; or the last 5
spr .tail 5 |display
```

&nbsp;

### More general Rye functions

Here are some more general functions that also work with tables. They all view table rows as collection items.


```lisp
spr .first
; returns a first Table Row

spr -> 0
; the same

spr .last
; returns a last Table Row

for spr { -> "Name" |print }
; for loop over the rows that prints:
;  Vladimir Harkonnen
;  Big brother
;  RDA Corporation
;  ...

spr .fold 'acc 0 { -> "Species" |= "AI" |either { acc + 1 } { acc } }
; a little convoluted way to count the number of AI's in a table
; using HOF-like function fold
; returns 5

```
&nbsp;

For the next section we will define a simple function that will sample 3 rows from the table and display those.

```lisp
dsp: fn1 { .head 3 |display }
```

> **fn1** (is a builtin function) that creates a function which accepts one argument and injects it into a block

&nbsp;

&nbsp;

![Cyberpunk docks](../cyberpunk_docks_2.webp)

## Endomorphic functions 1/2

Functions in this section have something in common. They accept a table and return a new table. And they don't change any state.

This means that you can, compose, apply and re-apply these function in any order and combination you need to.

While some verbiage is taken from SQL, because we've already know these terms, this is not trying to be a SQL substitute or equivalent. The goal is internal consistency with Rye itself.


### Selecting columns

Rye has convention of `?` at the end of the word meaning `get-(word)`. So `columns?` function gets you a **new** table with just specified columns.

```lisp
spr .columns? { "Name" "Species" } |dsp
; displays a new table with two columns
;
; | Name               | Species              |
; +-------------------------------------------+
; | Vladimir Harkonnen | Human                |
; | Big Brother        | Human (Entity)       |
; | RDA Corporation    | Human (Organization) |
```

> **Reminder** The dsp function we created above takes 3 random rows from the table and displays them.

### Filtering rows

We have multiple `where-*` functions that create a **new** table with just specified rows.

Why not just a single where function that accepts a block of Rye code? Because these run at Go's speeds.
It's better to apply Rye after we've already selected the rows of interest.

```lisp
spr .where-equal 'Species "Mutant"
; returns a new table where species is Mutant

x> spr .where-contains 'Species "Alien" |dsp
; displays a table where species field contains Alien 
;
; | Year | Villain        | Species | Keywords                                                   |
; +----------------------------------------------------------------------------------------------+
; | 2560 | The Covenant   | Alien   | Alliance Fanatical, Militaristic, Technologically Superior |
; | 1981 | The Overlords  | Alien   | Omniscient, Controlling, Mystical                          |
; | 2236 | The Puppeteers | Alien   | Cowardly, Manipulative, Intelligent                        |

spr .where-match 'Keywords regexp "(^|, )Inteligent(, |$)"
; returns a table where tags field matches regexp

spr .where-greater 'Year 2000
; returns a table year is greater than 2000

spr .where-lesser 'Year 2050
; returns a table year is lesser than 2050

spr .where-between 'Year 2000 2050
; returns a table where year is between the two values

spr .where-in 'Species { "Human" "AI" }
; returns a table where species is Human or AI

spr .where 'Keywords { .split "," |< 3 }              ;todo
; returns a table of rows with less than two keywords
```
&nbsp;

### Sorting rows

We can order table rows. This again returns a **new** table. This might be wasteful for big tables, so
on page 2/3 we will also look at `order-by!` that does this in-place.

```lisp
spr .order-by 'Year 'desc
; returns new table sorted by descending Year column

spr .order-by 'Name 'asc |head 3 |display
; returns a new table of top 3 rows sorted by names from A to Z
; 
; | Year | Name               | Species        | Keywords                            |
; +----------------------------------------------------------------------------------+
; | 2199 | Agent Smith        | AI Program     | Ruthless, Persistent, Corrupt       |
; | 10191| Vladimir Harkonnen | Human          | Scheming, Ruthless, Manipulative    |
; | 1984 | Big Brother        | Human (Entity) | Oppressive, Omnipresent, Tyrannical |
```
&nbsp;

### Limiting rows

We've seen them above, as they are regular Rye functions that work on collection types, also tables. But they
also return new tables.

```lisp
spr .head 3
; returns a new table with first 3 rows

spr .slice 10 3
; returns new table with 3 rows from tenth forward

spr .tail 1 |display
: displays a new table of just last row in xlsx file
; 
; | Year | Name        | Species         | Keywords                              |
; +------------------------------------------------------------------------------+
; | 2999 | Brain Slugs | Parasitic Alien | Control, Simplistic, Creepy           |
```
&nbsp;

### Putting it together

Lets try putting now these together.

We want to see the names, years and species columns of the human-ish adversaries ordered by year.

```lisp
spr .columns? { 'Name 'Year 'Species }
|where-contains 'Species "Human" 
|order-by 'Year asc 
|display
; 
; | Name               | Year   | Species              |
; +----------------------------------------------------+
; | Big Brother        | 1984   | Human (Entity)       |
; | Nolan Sorrento     | 2041   | Human                |
; | Precrime           | 2054   | Human (System)       |
; | Vicious            | 2071   | Human                |
; | The Giver Society  | 2074   | Human (Society)      |
; | RDA Corporation    | 2154   | Human (Organization) |
; | Vladimir Harkonnen | 10191  | Human                |
```
&nbsp;

Now we want to see earlyest 3 AI related adversaries and their keywords, but sorted by name. How do we get there.

We can filter by species, order that by year, take top 3, select just the columns we want and order by name.

```lisp
spr .where-contains 'Species "AI" 
|order-by 'Year 'asc 
|head 3 
|columns? { 'Name 'Keywords } 
|order-by 'Name 'asc 
|display
; 
; | Name     | Keywords                            |
; +------------------------------------------------+
; | HAL 9000 | Calculating, Logical, Fatal         |
; | Ultron   | Ambitious, Intelligent, Destructive |
; | VIKI     | Logical, Overbearing, Ruthless      |
```
&nbsp;

The fancy word "endomorphism" means that each of these functions accept a table and return a new table. This
means that we can combine and keep combining them freely. It's all apples to apples to apples ...

![Cyberpunk docks](../cyberpunk_docks_3.webp)

&nbsp;

## Next pages

We will find more endomorphic function on page 2 of this series. Functions like `add-column`, `group-by`, and `inner-join`.

We will also look into functions that do change tables **in-place**. The reason for using those could be big dataset and
optimisation. Or that we want to use table explicitly to store state in our program.

&nbsp;

&nbsp;

<!--
## Single column

### Column?

I noticed above that **Species** field held "Human", "Human (Entity)" and "Human (Organisation)". That's why 
I used `where-contains` above. I'm not sure what other values it has and if all make sense. So let's see them.

`column?` function returns values of a signle column as a *block*. And we can use may functions that work on blocks
with it then.

```lisp
spr .column? 'Species
|Unique
; returns
; { ... }
```
&nbsp;

### From Column to Block functions

Block is one of Rye's main building _blocks_ so we have a whole plethora of functions to work with them.

We are also interested what are the minimal and maximal years that our adversaries exist in.

```lisp
spr .column? 'Year :y
[ y .max , y .min ]
; returns a block
; { 12331 12324 }
```
&nbsp;

And when does the first AI appear

```lisp
spr .where-equal 'Species "AI"
|column? 'Year 
|min
; returns: 10230
```
-->

## Example from the wild

Github user losvedir made a little [language comparison challenge](https://github.com/losvedir/transit-lang-cmp/) a while back.

> The apps reads in the GTFS trips and stop times CSV-s. 
> There are roughly 120k trips and 3 mio stop times.
> Then set up a webserver that responds to /schedules/:route requests, and returns a JSON of all the trips for a route, 
> with included stop_times for each trip.

Solution in Rye uses almost only the functions we already looked at above, and it's nice to see how tables
fit with JSON and a web-server.

Couple of things to note. 

* Tables also support indexing (`add-indexes!`), which we use here. Without it the lookup for 3 million stop times for each
trip would simply take too long. With, it works promptly.
* We use function `add-column` which we will learn about in the next page. In short, it uses existing values of a row to generate a new column.
* Tables cells can also hold tables. That's how we return nested JSON of the stop times.

```lisp
; load data
data: context {
	load\csv %trips.big.csv |add-indexes! { route_id } :trips
	load\csv %stop_times.big.csv |add-indexes! { trip_id } :stop-times
} 

; query data and build response
build-response: fn { route } {
	data/trips
	|where-equal 'route_id route
	|columns? { 'trip_id 'service_id 'route_id }
	|add-column 'schedules { trip_id } {
		data/stop-times
		|where-equal 'trip_id trip_id
		|columns? { 'stop_id 'arrival_time 'departure_time }
	}
}

; helper functions
write-ok: fn { d w } { set-content-type w "application/json" , write w to-json d }
get-route: fn1 { .url? .path? .submatch?* regexp "/.*/(.*)" }
handle-schedules: fn { w r } { r .get-route |build-response |write-ok w }

; start server
http-server print ":4000"
|handle "/schedules/" ?handle-schedules
|serve
```
&nbsp;

Below, we can see the screenshot of calling the API with our browser. On the right, we also see required Rye code (~20 lines) and 
required Typescript code (~150 lines). Go and Rust solutions are even longer (~200 lines).

Rye is less verbose than many other the languages, but no language can be that less verbose on its own. In general, lines of code are not
that important, but I am trying to make a point here.

![Transit comparisson](../transit_comparisson.png)

&nbsp;

In my opinion, Rye takes so much less code
to do this **because of the table value type**. It has much better **problem-structure-fit** for this case. 
Because of it, our solution is quite declarative, using some basic operations over tabular data. 
That's why I think it's not shorter and more complex, but shorter and simpler at the same time. 

Now I will quote what I wrote right at the top of this page, to drive the point home :)

> You can not have a higher level (closer to human) langauge, without also higher level (closer to human) value types. These then enable higher level operations on information you are working with.

&nbsp;

_I am preparing Page 2 of the Table cookbook. Meanwhile, you can check the [rest of the site](https://ryelang.org). Rye [Reddit group](https://www.reddit.com/r/ryelang/) is currently 
the best way to keep up with Rye updates. And of course there is the [Github](https://github.com/refaktor/rye)._
