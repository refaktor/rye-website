---
title: "Date and time*"
weight: 50
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Functions for working with dates and time."
mygroup: true

---

*work-in-progress*

## Now


```clojure
x> now
[Time: .........]
```

## Components of Datetime

```clojure
now :n

; convention: ? at the end of the word (noun) means get-noun 

n .year?

n .month?

n .day?

n .hour?

n .minute?

n .second?
```

## Datetime math


```clojure
now :n

; convention: ? at the end of the word (noun) means get-noun 

n + 10 .seconds

n + 10 .minutes

n - 5 .hours

n + 1 .days

( n + 5 .hours ) > n + 5 .seconds  ; or

n + 5 .hours |> n + 5 .seconds

```

## Difference


```clojure
now :n1

; wait a little

now :n2

n1 - n2 :nd

print nd / 1 .seconds " seconds passed between"
```
