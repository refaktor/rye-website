---
title: "Date and time"
weight: 50
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Functions for working with dates and time."
mygroup: true

---

*work-in-progress*

## Now

Get current date-time.

```clojure
x> now
[Time: 2025-11-05 01:06:22]
```

## Components of Datetime

```clojure
x> now :n
[Time: 2025-11-05 01:08:53]
x> n .year?
[Integer: 2025]
x> n .month?
[Integer: 11]
x> n .day?
[Integer: 5]
x> n .hour?
[Integer: 1]
x> n .minute?
[Integer: 8]
x> n .second?
[Integer: 53]
```

## Datetime math

Simple calculation with date-time and comparisson.

```clojure
x> now :m
[Time: 2025-11-05 01:10:59]
x> m + 10 .seconds
[Time: 2025-11-05 01:11:09]
x> m + 10 .minutes
[Time: 2025-11-05 01:20:59]
x> m - 3 .hours
[Time: 2025-11-04 22:10:59]
x> m - 7 .days
[Time: 2025-10-29 01:10:59]
x> ( n + 5 .minutes ) > n + 50 .seconds
[Boolean: true]
```

## Difference

You can substract time to get the difference

```clojure
x> n: now
[Time: 2025-11-05 07:26:28]
; slowly count to ten
x> m: now
[Time: 2025-11-05 07:26:38]
x> m - n :mn
[Integer: 10591]
x> print ( mn / 1 .seconds ) ++ " seconds passed between"
; Prints: 
; 10.591 seconds passed between
```

## More functions

Look at [function reference](https://ryelang.org/info/base.html#heading-Date%20and%20Time%20) for more Date and Time related functions.
