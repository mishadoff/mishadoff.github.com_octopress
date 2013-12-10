---
layout: post
title: "Clojure Design Patterns"
date: 2013-12-11 00:09
comments: true
categories: [clojure, programming, java]
published: false
---

Overview of Design Patterns from the
[classic book](http://en.wikipedia.org/wiki/Design_Patterns) by Gang of Four. Some attempts to implement all of them in Clojure. 

<!-- more -->

**Disclaimer:** *Most patterns are easy to implement because
we use dynamic typing, functional programming and, of course,
Clojure. Some of them look ugly. It's okay.*

> Our programming language is fucked up.
>
> That's why we need design patterns.

We have three problems in programming:

- **create** an object for representing some real-world entity
- select an appropriate data **structure** for manupalating such objects
- define set of functions (**behaviour**) for such objects.

Patterns solve all these problems([*](#prog)):

- **Creational Patterns** help to create objects.

  Customer not interested in expensive `new Object()`
  (*what is the "new", should I pay for it?*),
  but `ImportantBusiness.getInstance()` looks great.
  
- **Structural Patterns** help to maintain relationships between objects.

  It's all about composition.

- **Behavioral Patterns** help to define set of functions.

  *What is "set of functions" you talking about?*
  Behavior sounds official.  

<a id="prog"></a>
(\*) *Programming is just one big problem: regexps, off-by-one errors, threads, timezones, cache invalidation and another million of topics everybody involved everyday, but this article about patterns, so we assume there are just 3 problems, and patterns solve them all.*

# Part I. Behavioral

## Command

"Encapsulates information needed to call a method
at a later time"

*Example:* `java.lang.Runnable`

Guess, what information do you need to call a method?

## Strategy

## 
