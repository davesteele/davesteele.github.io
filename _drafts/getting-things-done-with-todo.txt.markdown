---
layout: post
title:  "Getting Things Done with Todo.txt"
# date:   2016-4-23 12:34:00
categories: gtd
excerpt_separator: <!--more-->
---

## How I Configured Todo.txt for GTD

Theoretically, I've been following David Allen's
[Getting Things Done](https://gettingthingsdone.com/) (GTD) practices for over
ten years. In reality, my ToDo list has lain fallow for most of that period.
GTD requires the ability to see your upcoming work through a number of linked
views. I've never found a set of tools that let me navigate between those views
easily enough, and to make that ability available wherever I am.

After spending some time thinking about how to build the "right" tool from scratch,
I came to realize that my evolving concept looked very much like
[Todo.txt](http://todotxt.org/), with some extensions. That suggests the
possibilty of using an established ecosystem of tools to solve my problem. Huzzah!

In short, here is how I've configured Todo.txt to handle GTD my way. It involves a small,
compatible extension to the todo.txt file format, and a small script to perform
GTD-specific cleanup on that file. I'll also touch on the applications I am using.

<!--more-->

A quick note on environment - I run these tools on a mixture of Linux and Android
systems. While this should also work fine on Mac, Windows, or iOS platforms,
I've not tried it.

## GTD Extensions to todo.txt

My #1 goal in this exercise was to have a free form text file that serves as the
GTD project reference, with @context-specific todo lists generated automatically from
that file.

Here is an example of a GTD-ized todo.txt file:

    # PetCare
    #
    # Fred's Pet Emporium is at 555-555-5555

    Buy a cat from Fred @errands +PetCare
    Buy cat food @errands +PetCare
    Feed the cat @pending

    Reverse all the toilet paper rolls @home

    # WeeklyReview
    #
    
    Perform the Weekly Review @computer t:2018-09-01 rec:1w
    
    # X-None
    #

    Browse cat pictures on Reddit @computer +ImproveHappiness

Todo.txt applications recognize a set of [rules](https://github.com/todotxt/todo.txt)
for the format of the task file. Some have 
[extended](https://fossil.mpcjanssen.nl/r/simpletask-android/doc/trunk/app/src/main/assets/index.en.md) the format. For GTD purposes, I've added two additional small
extensions.

First, comments are allowed, identified by a leading '#'. Comments are used to provide
project-level information. Comments do not define specific tasks.

A comment consisting of a single word is defined as a project header. The word
is the project name. Tasks following a particular project header will default
to that project, unless they are explicitly set to another.

In the todo.txt file, there should be a special project, called "X-None", which is
used to collect tasks as they are entered externally, for eventual disbursement
to their home projects.


A GTD task is a line of text in todo.txt which has a context specified,
identified by a word with '@' as the first character (e.g. '@phone'). The
project is specified by a keyword with a leading '+' (e.g. '+WeeklyReview'). 

I use the '@pending' context to identify tasks that are later steps in a
multi-step process. Normally, in day-to-day activities, '@pending' tasks are
ignored. The context is manually changed to an active context when the proceeding
tasks are complete.


## The GTD cleanup script


## Configuring the Apps


## The Weekly Review using Todo.txt
