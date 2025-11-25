---
title: "Options*"
weight: 300
_date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Options of deploying your Rye programs"
mygroup: true
---

*heavily-work-in-progress*

## Rye binary

Rye interpreter comes as a statically compiled binary, so you in general need a **single executable file** on your system
and Rye runs. 

Rye runs on Linux, MacOS and Windows and it's been tested on Android so far.

## Per project binary

Current idea is, or at least I do it like this, that I have one general `rye` installed that I can invoke at any time and at any location. 

But If I tackle concrete and specific project I build a project specific binary and keep it local to the project. 

It's all experimental but I made some utilities to run the local rye, to define modules it includes and excludes and rebuild it automatically.
Things are still in flux, so more about this later.

## Batteries included

Main Rye interpreter already comes with multiple libraries included. We will make functions so you can explore the modules in the binary at hand
when in Rye console.

Before you had to define which all modules you wanted in your build of Rye, now a lot of them are included by default and you can use build flags
to exclude them. A little more rare ones, or problematic ones, or problem/OS specific ones still need to be included. This will systemize yet, as 
we use it all more and see how it works in real world.

## External extensions

In case of big libraries, like a GUI framewok Rye is extended externally. Examples of this are Fyne, Gio and Ebitengine. You basically don't include
the huge library into Rye, but you make a custom binary that has full Rye with batteries included into your BIG library extension. It was an experiment
but it functions quite OK for now. Also separates concerns and projects with very different goals and dependencies.

Because of this we have Rye-Fyne, Rye-Gio and Rye-Ebitengine as separate projects and repositories. With very little Go code these project offer all
the console behavior of normal Rye. 

*we will document how you can make your own extension or embedd Rye into your own Go program*

## Rye binary behaviour

You can read all about how you can invoke Rye, what flags and commands you can use, etc on the [Rye binary cookbook page](../../binary_console/executable/).

## Embed main build

Again, to be improved but there is a proof of concept way to embed your main.rye with a Rye binary into a separate standalone binary. You can then
distribute your whole Rye program as one executable, no additional script needed (and it can't invoke custom scripts).

Besides simpler deployment to users this might also have some safety benefits on the client and even, I did not expect that at first, on the server side. 
For example AppArmour rules can generally be set per binary, so if Rye is a binary that can run any script it sorts of defeats that purpose or at least
makes it much more complicated and uncertain.

You can create embedded binaries with Rye fyne also. So you get an executable that directly opens the GUI.

Current work in progress process goes like this.


Let's say you are in your project's directory. You have a main.rye file here. Currently this works just for a single main.rye, but it will get extended.

There is a script ryel (rye local) that does this, but let's see which commands exactly it does

```
# You define RYE_HOME variable
export RYE_HOME=/home/user/rye

# You define flags into ryel.mod file
cat > ryel.mod
no_sqlite
no_terminal

# To build a local rye binary you call
ryel build
# Or 
$RYE_HOME/bin/rye $RYE_HOME/util/ryel.rye build

# To run local rye binary with local main.rye
ryel .
# Or
./rye .

# To build a local rye binary that embeds main.rye
ryel build embed_main
# Or
$RYE_HOME/bin/rye $RYE_HOME/util/ryel.rye build embed_main
```

There are changes being made so you can do this also for Rye-Fyne, Rye-Gio, ...

## Building apk from Rye app

*to-be-written*
