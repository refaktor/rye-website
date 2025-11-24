---
title: "Rye Tables 2/2"
weight: 52
___date: 2025-05-21T09:36:00+01:00
draft: false
group: true
summary: "Advanced functions for working with table value type."
mygroup: true
toc: true
---

&nbsp;

If you haven't you really **shoud read the [first part](tables.md)** of our tables cookbook. There we explored the basics of creating, loading, and manipulating tables in Rye. 

Now, we'll look into more advanced operations that make tables even more powerful.

<!-- ![Data analysis](../data_analysis.webp) -->

## Symmetric functions 2/2

For the lack of better word,symmtre - functions that accept a table and return a new table. Here we'll continue with more powerful symmetric functions.

### Adding rows

While we can modify tables in-place (which we'll see later), we can also create new tables with additional rows:

```lisp
books: table { 'title 'author 'year } 
       { "Neuromancer" "William Gibson" 1984 
         "Snow Crash" "Neal Stephenson" 1992 }

; add-row returns a new table with the added row
new-books: books .add-row { "The Diamond Age" "Neal Stephenson" 1995 }
; 
; | title          | author          | year |
; +----------------------------------------+
; | Neuromancer    | William Gibson  | 1984 |
; | Snow Crash     | Neal Stephenson | 1992 |
; | The Diamond Age| Neal Stephenson | 1995 |
```

For adding multiple rows at once, use `add-rows` (without the exclamation mark) to create a new table:

```lisp
; Add multiple rows to a table (returns a new table)
expanded-books: books .add-rows { "Cryptonomicon" "Neal Stephenson" 1999
                                  "Anathem" "Neal Stephenson" 2008 }

; You can also get rows from one table and add them to another
stephenson-books: books .where-equal 'author "Neal Stephenson"
rows: stephenson-books .get-rows

new-collection: table { 'title 'author 'year } 
                { "Dune" "Frank Herbert" 1965 }
combined-collection: new-collection .add-rows rows
```

### Adding columns

One of the most powerful features of tables is the ability to generate new columns from existing data. This allows you to transform and enrich your data without leaving the table paradigm:

```lisp
books: table { 'title 'year } 
       { "Neuromancer" 1984 
         "Snow Crash" 1992
         "The Diamond Age" 1995 }

; Add a column showing the age of each book
current-year: 2025
books-with-age: books .add-column 'age { year } { current-year - year }
; 
; | title          | year | age  |
; +-----------------------------+
; | Neuromancer    | 1984 | 41   |
; | Snow Crash     | 1992 | 33   |
; | The Diamond Age| 1995 | 30   |
```

The `add-column` function is particularly powerful because it lets you:
1. Access values from existing columns for each row
2. Apply arbitrary transformations using Rye code
3. Create the new column based on those calculations

You can even use regex replacement to generate a new column:

```lisp
names: table { 'full_name } 
       { "Arthur C. Clarke" 
         "Isaac Asimov"
         "Philip K. Dick" }

; Extract last names using regex
names-with-last: names .add-column 'last_name { full_name } 
                 { regexp "^.*\s+([^\s]+)$" "$1" }
; 
; | full_name        | last_name |
; +-----------------------------+
; | Arthur C. Clarke | Clarke    |
; | Isaac Asimov     | Asimov    |
; | Philip K. Dick   | Dick      |
```

This is a powerful way to transform data without having to write complex loops or use multiple functions.

### Filtering methods 2/2

In the first part, we saw filtering functions like `where-equal`, `where-contains`, and `where-greater`. Here are more specialized filtering functions that give you even more control:

```lisp
; Table of alien species
aliens: table { 'species 'homeworld 'intelligence 'threat_level }
        { "Xenomorph" "Unknown" 7 "Extreme"
          "Predator" "Yautja Prime" 9 "High"
          "Na'vi" "Pandora" 8 "Low"
          "Vulcan" "Vulcan" 10 "Low"
          "Klingon" "Qo'noS" 8 "Medium" }

; Filter with pattern matching
aliens .where-match 'species regexp "^[PX]"
; Returns species starting with P or X

; Filter with negated string containment
aliens .where-not-contains 'homeworld "a"
; Returns species whose homeworld doesn't contain "a"

; Filter with value in a set
aliens .where-in 'threat_level { "High" "Extreme" }
; Returns species with high or extreme threat levels

; Filter with inclusive range
aliens .where-between\inclusive 'intelligence 8 9
; Returns species with intelligence of 8 or 9
```

Tables can contain missing data represented as void values. You can filter for these specifically:

```lisp
crew: table { 'name 'position 'specialty } 
      { "Ripley" "Lieutenant" "Command"
        "Dallas" "Captain" _
        "Kane" "Executive Officer" _
        "Ash" "Science Officer" "Xenobiology" }

; Find crew members with missing specialties
crew .where-void "specialty"
; 
; | name   | position          | specialty |
; +--------------------------------------+
; | Dallas | Captain           | _        |
; | Kane   | Executive Officer | _        |
```

### Inner join

One of the most powerful operations is joining tables together. The `inner-join` function combines two tables based on matching values in specified columns, keeping only rows that have matches in both tables:

```lisp
; Table of characters
characters: table { 'id 'name 'species } 
            { 1 "Deckard" "Human"
              2 "Roy Batty" "Replicant"
              3 "Rachael" "Replicant"
              4 "Gaff" "Human" }

; Table of occupations
occupations: table { 'id 'occupation } 
             { 1 "Blade Runner"
               2 "Combat Model"
               4 "Police Officer" }

; Inner join - only keeps rows with matches in both tables
characters .inner-join occupations 'id 'id
; 
; | id | name      | species   | id_2 | occupation    |
; +--------------------------------------------------+
; | 1  | Deckard   | Human     | 1    | Blade Runner  |
; | 2  | Roy Batty | Replicant | 2    | Combat Model  |
; | 4  | Gaff      | Human     | 4    | Police Officer|
```

Notice that Rachael (id 3) is not in the result because there's no matching occupation.

### Left join

The `left-join` function is similar to `inner-join`, but it keeps all rows from the left table, even if they don't have matches in the right table:

```lisp
; Left join - keeps all rows from the left table
characters .left-join occupations 'id 'id
; 
; | id | name      | species   | id_2 | occupation    |
; +--------------------------------------------------+
; | 1  | Deckard   | Human     | 1    | Blade Runner  |
; | 2  | Roy Batty | Replicant | 2    | Combat Model  |
; | 3  | Rachael   | Replicant | _    | _             |
; | 4  | Gaff      | Human     | 4    | Police Officer|
```

Notice that Rachael (id 3) is included in the result, but with void values for the columns from the right table.

### Group by

The `group-by` function allows you to aggregate data based on common values. This is useful for summarizing data and calculating statistics.

Here's a simple example:

```lisp
; Table of books by genre
books: table { 'title 'author 'genre 'year }
       { "Neuromancer" "William Gibson" "Cyberpunk" 1984
         "Snow Crash" "Neal Stephenson" "Cyberpunk" 1992
         "Dune" "Frank Herbert" "Science Fiction" 1965
         "Foundation" "Isaac Asimov" "Science Fiction" 1951
         "The Shining" "Stephen King" "Horror" 1977 }

; Group by genre and count books in each genre
books .group-by 'genre { 'genre count }
; 
; | genre           | genre_count |
; +-----------------------------+
; | Cyberpunk       | 2           |
; | Science Fiction | 2           |
; | Horror          | 1           |
```

For more complex aggregations, you can calculate multiple statistics at once:

```lisp
; Table of space colonies
colonies: table { 'name 'planet 'population 'founded }
          { "New Terra" "Mars" 50000 2130
            "Olympus" "Mars" 75000 2145
            "Titan Base" "Titan" 12000 2160
            "Europa Station" "Europa" 8000 2155
            "Phobos Mining" "Phobos" 3000 2140
            "Deimos Research" "Phobos" 1500 2150 }

; Group by planet and calculate statistics
colonies .group-by 'planet { 'planet count 'population sum 'population avg 'founded min }
; 
; | planet | planet_count | population_sum | population_avg | founded_min |
; +----------------------------------------------------------------------+
; | Mars   | 2            | 125000         | 62500          | 2130        |
; | Titan  | 1            | 12000          | 12000          | 2160        |
; | Europa | 1            | 8000           | 8000           | 2155        |
; | Phobos | 2            | 4500           | 2250           | 2140        |
```

The `group-by` function supports several aggregation operations:
- `count` - Count the number of rows
- `sum` - Sum the values in a column
- `avg` - Calculate the average of values in a column
- `min` - Find the minimum value in a column
- `max` - Find the maximum value in a column

### Creating tables from other data structures

You can create tables from lists or blocks of dictionaries using the `to-table` function:

```lisp
; Create a list of dictionaries
ai-systems: list [
  dict { "name" "HAL 9000" "year" 1968 "status" "Decommissioned" }
  dict { "name" "SHODAN" "year" 1994 "status" "Active" }
  dict { "name" "GLaDOS" "year" 2007 "status" "Active" }
]

; Convert to a table
ai-table: to-table ai-systems
; 
; | name    | year | status         |
; +----------------------------------+
; | HAL 9000| 1968 | Decommissioned |
; | SHODAN  | 1994 | Active         |
; | GLaDOS  | 2007 | Active         |
```

This is particularly useful when working with data from APIs or when building data structures programmatically.

## Indexing for performance

For large tables, you can create indexes to speed up lookups. This is especially important when working with tables containing thousands or millions of rows:

```lisp
; Create a large table of space missions
missions: table { 'id 'mission 'year 'destination }
          { 1 "Apollo 11" 1969 "Moon"
            2 "Voyager 1" 1977 "Interstellar space"
            3 "Cassini" 1997 "Saturn"
            ; ... imagine hundreds more rows
          }

; Add indexes for faster lookups
missions .add-indexes! { destination year }

; Check which columns are indexed
missions .indexes?
; returns { "destination" "year" }

; Now lookups by destination or year will be much faster
missions .where-equal 'destination "Moon"
```

Indexes work by creating a map from column values to row indices, allowing for O(1) lookups instead of O(n) scans. This can dramatically improve performance for large tables.

## Modifying tables in place

All the operations we've seen so far follow Rye's functional approach - they don't modify the original data but instead return new tables with the changes applied. This is generally preferred as it makes code more predictable and easier to reason about.

However, sometimes we need to modify tables in place:
- When using tables as application state or data storage
- For performance optimization with very large tables
- When implementing stateful algorithms that require mutation

Rye follows a convention where functions that modify data in place end with an exclamation mark (!).

### Adding and removing rows in place

To add rows to an existing table without creating a new one:

```lisp
; Create a reference to a table
books-ref: ref table { 'title 'author 'year } 
               { "Neuromancer" "William Gibson" 1984 
                 "Snow Crash" "Neal Stephenson" 1992 }

; Add multiple rows to the reference (modifies in-place)
books-ref .add-rows! { "Cryptonomicon" "Neal Stephenson" 1999
                       "Anathem" "Neal Stephenson" 2008 }
; The original table is modified
```

To remove a row:

```lisp
; Remove the second row
books-ref .remove-row! 2
; The "Snow Crash" row is now gone from the original table
```

### Updating rows in place

You can update specific rows with new values:

```lisp
; Update the first row with a dictionary
books-ref .update-row! 1 dict [ "year" 1983 ]
; The year for "Neuromancer" is now 1983

; Update using a function
update-year: fn { row } { 
  row + [ "year" ( "year" <- row ) + 1 ] 
}
books-ref .update-row! 1 ?update-year
; The year for "Neuromancer" is now 1984 again
```

### Sorting in place

Instead of creating a new sorted table, you can sort the original:

```lisp
; Sort the table by year in ascending order
books-ref .order-by! 'year 'asc

; Sort by author in descending order
books-ref .order-by! 'author 'desc
```

### Modifying columns in place

You can rename columns without creating a new table:

```lisp
; Rename the "year" column to "published_year"
books-ref .rename-column! "year" "published_year"
```

These in-place operations are particularly useful when working with large datasets or when implementing algorithms that require stateful changes to data.

## Saving tables

In the first part, we saw how to load tables from various formats like CSV and Excel. Now let's look at how to save tables to these formats.

```lisp
; Save a table to CSV
planets: table { 'name 'type 'moons } 
         { "Earth" "Terrestrial" 1
           "Mars" "Terrestrial" 2
           "Jupiter" "Gas Giant" 79
           "Saturn" "Gas Giant" 82 }

planets .save\csv %planets.csv

; Save to TSV (tab-separated values)
planets .save\tsv %planets.tsv

; Save to Excel format
planets .save\xlsx %planets.xlsx
```

You can also save tables to BSON format, which preserves type information:

```lisp
; Save to BSON format (preserves types)
planets .save\bson %planets.bson

; Load from BSON
bson-planets: load\bson %planets.bson
```

Unlike CSV files, BSON preserves the data types of your values, so you don't need to use `autotype` when loading.

## Real-world example: Analyzing exoplanet data

Let's put these functions together in a more complex example using our exoplanet dataset:

```lisp
; Load exoplanet data
exoplanets: load\csv %exoplanets.csv |autotype 0.95

; Display the first few rows
exoplanets .head 3 |display
; 
; | name        | type       | mass  | radius | temp  | habitable | discovery_year |
; +-------------------------------------------------------------------------------+
; | Kepler-186f | Terrestrial| 1.17  | 1.11   | 188   | true      | 2014           |
; | TRAPPIST-1e | Terrestrial| 0.77  | 0.92   | 251   | true      | 2017           |
; | Proxima Cen b| Terrestrial| 1.27  | 1.08   | 234   | true      | 2016           |

; Find potentially habitable planets discovered in the last 5 years
recent-habitable: exoplanets 
                  |where-equal 'habitable true
                  |where-greater 'discovery_year 2020
                  |order-by 'mass 'asc

; Calculate Earth Similarity Index (fictional formula)
recent-habitable .add-column 'esi { mass radius temp } {
    mass-factor: 1 - abs (mass - 1)
    radius-factor: 1 - abs (radius - 1)
    temp-factor: 1 - (abs (temp - 288) / 288)
    (mass-factor + radius-factor + temp-factor) / 3
}

; Group planets by type and calculate statistics
exoplanets .group-by 'type { 
    'type count 
    'mass avg 
    'radius avg 
    'temp avg 
    'habitable count
}
|order-by 'type_count 'desc
|display
```

### Finding similar planets using vector operations

Rye has powerful vector operations that we can use to find similarities between planets. Let's use these to find exoplanets that are most similar to Earth:

```lisp
; Define Earth's properties as our reference
earth-properties: vector [ 1.0 1.0 288.0 ]  ; mass, radius, temperature

; Extract properties from each exoplanet and calculate similarity to Earth
exoplanets .add-column 'properties { mass radius temp } {
    vector [ mass radius temp ]
}
|add-column 'earth_similarity { properties } {
    cosine-similarity? properties earth-properties
}
|order-by 'earth_similarity 'desc
|head 5
|display
; 
; | name        | type       | mass  | radius | temp  | ... | earth_similarity |
; +-----------------------------------------------------------------------------+
; | Kepler-452b | Terrestrial| 1.03  | 1.05   | 265   | ... | 0.9998          |
; | TOI-700 d   | Terrestrial| 1.07  | 0.95   | 268   | ... | 0.9997          |
; | Teegarden b | Terrestrial| 1.05  | 1.02   | 250   | ... | 0.9996          |
; | Kepler-1649c| Terrestrial| 1.06  | 1.06   | 234   | ... | 0.9995          |
; | Luyten b    | Super-Earth| 2.89  | 1.35   | 259   | ... | 0.9993          |
```

We can also use correlation to find planets with similar patterns of properties, even if the absolute values differ:

```lisp
; Find planets with correlated properties to a reference planet
reference-planet: exoplanets .where-equal 'name "Kepler-186f" |first

; Create a vector from the reference planet's properties
reference-vector: vector [ 
    reference-planet -> "mass" 
    reference-planet -> "radius" 
    reference-planet -> "temp" 
]

; Calculate correlation with the reference planet
exoplanets .add-column 'correlation { mass radius temp } {
    planet-vector: vector [ mass radius temp ]
    correlation planet-vector reference-vector
}
|order-by 'correlation 'desc
|head 5
|display
; 
; | name        | type       | mass  | radius | temp  | ... | correlation |
; +-------------------------------------------------------------------------+
; | Kepler-186f | Terrestrial| 1.17  | 1.11   | 188   | ... | 1.0000      |
; | Kepler-442b | Terrestrial| 1.33  | 1.12   | 233   | ... | 0.9876      |
; | Proxima Cen b| Terrestrial| 1.27  | 1.08   | 234   | ... | 0.9854      |
; | TRAPPIST-1g | Terrestrial| 1.34  | 1.13   | 199   | ... | 0.9812      |
; | Kepler-62f  | Terrestrial| 1.41  | 1.41   | 208   | ... | 0.9743      |
```

We can also analyze the variability of different planet types using standard deviation:

```lisp
; Group planets by type and calculate standard deviation of properties
exoplanets .group-by 'type { 'type count }
|order-by 'type_count 'desc
|head 4
|fold 'results [] { ::type-row ::results
    type-name: type-row -> "type"
    planets-of-type: exoplanets .where-equal 'type type-name
    
    ; Get mass, radius and temp as vectors
    masses: planets-of-type .column? 'mass |vector
    radii: planets-of-type .column? 'radius |vector
    temps: planets-of-type .column? 'temp |vector
    
    ; Calculate standard deviations
    mass-std: masses .std-deviation?
    radius-std: radii .std-deviation?
    temp-std: temps .std-deviation?
    
    ; Add to results
    + results dict {
        "type" type-name
        "count" (type-row -> "type_count")
        "mass_std" mass-std
        "radius_std" radius-std
        "temp_std" temp-std
    }
}
|to-table
|display
; 
; | type        | count | mass_std | radius_std | temp_std |
; +--------------------------------------------------------+
; | Terrestrial | 21    | 1.23     | 0.24       | 512.8    |
; | Super-Earth | 14    | 4.31     | 0.51       | 587.3    |
; | Gas Giant   | 10    | 254.7    | 4.63       | 823.5    |
; | Neptune-like| 7     | 7.82     | 2.74       | 278.4    |
```

This example demonstrates how you can:
1. Load and clean data
2. Use vector operations to find similarities between data points
3. Calculate correlations between different entities
4. Analyze variability using standard deviation
5. Group and aggregate data
6. Sort and display results

All of this is done with a clean, functional pipeline that's easy to read and modify.

## Conclusion

Tables in Rye provide a powerful way to work with structured data. The endomorphic functions we've explored in this second part enable complex operations like joining datasets, aggregating data, and transforming tables in various ways.

The ability to chain these operations together creates a powerful data processing pipeline that's both expressive and efficient. Whether you're analyzing scientific data, processing business information, or just organizing your personal collection, Rye's table functions give you the tools you need.

In the next part, we'll explore functions that modify tables in-place, as well as more advanced integration with other Rye features.

![Data visualization](../data_visualization.webp)

&nbsp;

_Check out the [rest of the site](https://ryelang.org) for more Rye documentation. Join the [Reddit group](https://www.reddit.com/r/ryelang/) to keep up with Rye updates, or visit the [GitHub repository](https://github.com/refaktor/rye)._
