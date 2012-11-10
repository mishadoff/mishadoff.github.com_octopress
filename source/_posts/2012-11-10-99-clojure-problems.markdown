---
layout: post
title: "99 Clojure Problems"
date: 2012-11-10 16:20
comments: true
categories: [clojure, programming]
published: false
---

There is a popular article [99 Prolog Problems](https://sites.google.com/site/prologsite/prolog-problems) that describes
how Prolog solves various small programming problems.
There are couple of remakes in [Lisp](http://www.ic.unicamp.br/~meidanis/courses/mc336/2006s2/funcional/L-99_Ninety-Nine_Lisp_Problems.html),
[Scala](http://aperiodic.net/phil/scala/s-99/), [Haskell](http://www.haskell.org/haskellwiki/H-99:_Ninety-Nine_Haskell_Problems) and more.
I found articles of such type extremely useful for programming language understanding, as they show solution for common real-world problems.

In this article, codename **CLJ-99**, I provide my solutions to these problems, reuse as much as possible standard `clojure.core` functions.

Improvements are welcome!

<!-- more -->

Solutions will be grouped by topic. To not reinvent wheel, I use the same groups as in original article.

Complexity introduced by stars: \* - simple, \*\* - medium, \*\*\* - hard.

Ready? Go!

## Lists

# P01. (*) Find the last element of a list.

(last [1 2 3 4]) => 4

P02 (*). Find the last but one element of a list.

(last (butlast [1 2 3 4])) => 3

P03 (*). Find the Kth element of a list.

(nth [1 2 3 4] 2) => 3

P04 (*). Find the number of elements of a list.

(count [1 2 3 4]) => 4

P05 (*). Reverse a list.

(reverse [1 2 3 4]) => [4 3 2 1]

P06 (*). Find out whether a list is a palindrome.
A palindrome can be read forward or backward; e.g. [x,a,m,a,x]

(= [1 2 1] (reverse [1 2 1])) => true

P07 (\*\*). Flatten a nested list structure.
Transform a list, possibly holding lists as elements into a 'flat' list by replacing each list with its elements (recursively).

(flatten [1 [2 [3 4] 2] 1]) => [1 2 3 4 2 1]

P08 (**). Eliminate consecutive duplicates of list elements.
If a list contains repeated elements they should be replaced with a single copy of the element. The order of the elements should not be changed.

TODO
