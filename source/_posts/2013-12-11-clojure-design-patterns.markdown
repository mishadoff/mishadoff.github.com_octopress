---
layout: post
title: "Clojure Design Patterns"
date: 2013-12-11 00:09
comments: true
categories: [clojure, programming, java]
published: false
---

Overview of Design Patterns from the
[classic book](http://en.wikipedia.org/wiki/Design_Patterns) by Gang of Four.
Some attempts to implement all of them in Clojure. 

<!-- more -->

**Disclaimer:** *Most patterns are easy to implement because
we use dynamic typing, functional programming and, of course,
Clojure. Some of them look ugly. It's okay.*

---
<br/>

> Our programming language is fucked up.
>
> That's why we need design patterns.

We have three problems in programming:

- First of all, we have to **create** an object for representing some real-world entity
- then, select an appropriate data **structure** for manupalating such objects
- and finally, define set of functions (**behaviour**) for such objects.

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
(\*) *Programming is just one big problem: regexps, off-by-one errors,
threads, timezones, cache invalidation and million of topics,
but this article about patterns, so we assume there are just 3 problems,
and patterns solve them all.*

---

# Part I. Behavioral

## Command

"Encapsulates information needed to call a method
at a later time"

*Example:* `java.lang.Runnable`

Guess, what information do you need to call a method?

Correct, *object and method name* in OOP world or just *function* in FP.
And a way to apply it.

``` clojure
(defn execute-command [command]
  (command))
```

This is how abstract command pattern looks like. `command` is a function,
and `(command)` call to that function.

**Task:** Implement two commands for login and logout.

*Parameters?* Not a problem.

One approach we can wrap our function with arguments to
anonymous no-arg function.

``` clojure
(execute-command #(login "user" "password"))
(execute-command #(logout "user"))
```

Another approach to implement `execute-command` with varargs.
This way we can avoid unnecessary wraping.

``` clojure
(defn execute-command [command & args]
  (apply command args))

(execute-command login "user" "password")
(execute-command logout "user")
```

*History?* Not a problem.

We setting up a list to save our commands.

``` clojure
(def history (atom [])) 

(defn execute-command [command & args]
  (swap! history conj [command args])
  (apply command args))
```

Every call to `execute-command` will modify `history` vector
that you can inspect later and restore all activites.

**Conclusion:** Command is just a function.

## Strategy

"Algorithm's behavior can be selected at runtime"

*Example:* `Collections.sort(list, comparator)`

For this particular example `sort` is an **algorithm**,
list is **data** to be sorted (algorithm's input),
and **comparator** runtime behaviour.

**Task:** Implement a function to check if collection
is sorted according to comparator.

``` clojure
(defn sorted? [coll comp]
  (apply comp coll))

(sorted? [1 2 3] <) => true
(sorted? [1 8 4] <) => false
(sorted? [7 6 5] >) => true
```

Here is `<` and `>` are binary predicates
(two-args functions return boolean) that define
algorithm's behaviour.

*But behaviour can be not just one function?* Sure.

``` clojure
(defn sorted? [coll context]
  (let [{:keys [fun message]} context]
    (print message)
    (apply fun coll)))

(sorted? [1 2 3] {:fun < :message "Less"}) => true
```

We passed "context" as a behaviour.
It can contain functions and data as well.

**Conclusion:** Strategy is just a function accepts another function.

## State

"Encapsulate varying behavior for the same routine based on an object's state"

*Example:* Not found in JDK

State pattern is very close to the Strategy.
It achieves the same goal using different mechanism:
instead of passing behavior to the method, we support some internal
state for the object, which affects method behaviour.
Moreover, successive calls can change object's state.

**Task:** Implement switch for the lightbulb

From clojure perspective it can be implemented the same way as strategy pattern.
But we will use *different mechanism* - multimethods.

Clojure is immutable, so let's do it correct -
pass state to the `switch` function, which do the work
and returns new state.

```
(defmulti switch :state)

(defn- flip [state]
  (->> (:state state)
       (get {:on :off :off :on})
       (assoc state :state)))

(defmethod switch :on [state]
  (println (:obj state) "is ON")
  (flip context))
 
(defmethod switch :off [state]
  (println (:obj state) "is OFF")
  (flip context))
```

Lotta code, but idea is simple.
We declare multimethod `switch` that use `:state` as dispatch function.
Then we define first function `switch` dispatched by value `:on`.
If call to switch sees value `:on` for key `:state` in state object,
this function is executed. The same applies for `:off` implementation. 

This code turn switch 10 times.

``` clojure
(def initial-state {:state :on :obj "Lightbulb"})
(take 10 (iterate switch initial-state))
```

But if we save state as an atom, code will become very similar to the strategy pattern.

``` clojure
(def state (atom {:state :on :obj "Lightbulb"}))

(defn switch []
  (let [{:keys [state obj]} @state]
    (println obj "is" state) 
    (swap! context flip)))

(repeatedly 10 switch)
```

**Conclusion:** State pattern is the same as Strategy.

## Template Method

"It lets redefine certain steps of an algorithm without changing the algorithm's structure"

*Example:* `java.util.AbstractList`

Assume we have an algorithm to move character in some RPG world.
All characters have a lot in common:

- they looking valuable items in chests
- if they found better artifact, replace their artifact
- if they encountered an enemy, attack or flee, depending on enemy strength
- if they found a questgiver, accept quest

But behavior between specific classes can vary depending on skills.
For example, most characters ignore closed chest, but thief can lockpick it,
most characters accept melee combat, but hunters and wizards try to keep distance, etc.

Abstract move for abstract character can be as follows

``` clojure
(defn move-to [loc]
  (cond
    (chest? loc) (let [chest (:chest loc)]
                   (if (open? chest) 
                     (take-from chest)
                     (do-nothing)))
    (artifact? loc) (let [art (:art loc)]
                     (if (better :art (my))
                       (replace :art)))
    (enemy? loc) (attack :melee (:enemy loc))
    (quest? loc) (accept quest)))
```

But we agreed some action can be overridden depending on character class.
We can extract them to parameters and provide default ones.

``` clojure
(defn move-to [loc & {keys [lock-chest att-type :or
                           {:lock-chest do-nothing :att-type :melee}]}]
  (cond
    (chest? loc) (let [chest (:chest loc)]
                   (if (open? chest) 
                     (take-from chest)
                     (lock-chest)))
    (artifact? loc) (let [art (:art loc)]
                     (if (better :art (my))
                       (replace :art)))
    (enemy? loc) (attack att-type (:enemy loc))
    (quest? loc) (accept quest)))
```

Then we will have different calls for different classes

``` clojure
(move-to loc) ;; Warrior
(move-to loc :lock-chest lockpick) ;; Thief
(move-to loc :att-type :ranged) ;; Hunter
```

You see? We just pass functions.

**Conclusion:** Template method is the same as Strategy

## Iterator
