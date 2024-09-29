---
title: 'Limiting effects*'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 300
summary: "In place changes are rare and visible, direct out of context changes are impossible."
mygroup: true
---

Rye as a language is very flexible, it tries to be very flexible in letting you define the procedure. But it tries to be much
more strict in terms of it's environment, especialy modifications of it.

## Visible in-place modifications

Most functions in Rye return values, and don't change them in place. In fact I would say we usually use just these functions and
you can do anything with just those functions.

But sometimes you want to specifically change some state in-place, so that must be possible too. To make it possible, but visible
all such function names must end with "!" or must include wordage "mod-" "set-".

*-convention is still somewhat being tested in practice-* 
