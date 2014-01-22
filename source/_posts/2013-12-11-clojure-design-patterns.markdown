---
layout: post
title: "Clojure Design Patterns"
date: 2013-12-11 00:09
comments: true
categories: [clojure, programming, java, story]
published: false
---

Story about classic [Design Patterns](http://en.wikipedia.org/wiki/Design_Patterns)
with attempts to implement them in Clojure.

<!-- more -->

**Disclaimer:** *Most patterns are easy to implement because
we use dynamic typing, functional programming and, of course,
Clojure. Some of them look wrong and ugly. It's okay.
All characters are fake, coincidences are accidental.*

---

# Angel of Dev

---  

*or how to write social network for cats in 23 weeks*

![PICTURE: ANGEL OF DEV]()

> Our programming language is fucked up.
> That's why we need design patterns.
> -- Anonymous

## Intro

**Pedro Veel** was modest Java Software Engineer in a
Leading European IT Service Provider company
called **Serpent Hill & R.E.E.**
The latest project he was working on, bank trading platform for
a large investment bank **Weats Inc.**, crashed due to
float numbers rounding error in production. 
Company has been bankrupted. 
Obviously, whole development team has been fired.

Pedro didn't regret too much about his job. He found 
that rounding bug earlier, and tried to prove its importance,
but QA lead decided that non-localized labels for Latin
language were the highest priority.

In a month he found a new job: in a startup company,
that got investments from business angel **Sven Tori**
few days ago for developing *"social network for cats"*.
They really tried to solve real-world problem,
and Pedro felt great about it. Moreover, he was only one
developer in the startup and could choose programming language.
Long time he wanted to try clojure in production.

Current team consists of four members:
project manager **Rage Man**, team lead **Mate Dale**,
beautiful receptionist **Terry P.** and our hero **Pedro**.

## Season I. Behavioral

Development started.

*Dale:* Pedro, we have to show investor our first demo on Friday.  
*Pedro:* What needs to be done?  
*Dale:* I suppose, registration, login and logout functionality.
Let me verify with Rage Man, but start working now.  
*Pedro:* Ok.  

Pedro never started new project alone (plus *clojure!*) and didn't know how to begin.
He googled the words *lein*, *ring*, *emacs* and started to code.

Googling login/logout functionality gives him a comment on forum for necromancers:
*"If you need dev help, summon Angel of Dev. Spell text is under the cut."*.

*Pedro:* What a bullshit.  

Four days gone and Dale came to Pedro.

*Dale:* Hi, Pedro. Yesterday we were at the theater with Terry P, I think I like her.  
*Pedro:* Great...  
*Dale:* What about our Friday's delivery?  
*Pedro:* Emm...  

Nothing was done. But Pedro has one day left.

*Pedro:* Will be in time.  
*Dale:* Excellent! Did I tell you are a great developer?  
*Pedro:* No.  
*Dale:* You are a great developer!  
*Pedro:* Thanks.  

Dale left the room.

Pedro was frustrated. He opened browser history and found link to the 
necromancers' forum. Looked at the empty room and yelled:

> Angel of Dev, while true I summon you!

Blue screen of death appeared on the monitor.
Some strange being climb out to the table.

*Pedro:* Who are you?  
*Vaine:* Hi, my name is Vaine. I'm an Angel of Dev.  
*Pedro:* An angel?  
*Vaine:* Not an angel, actually. I help developers. Do you need help?  
*Pedro:* I need to implement a register/login/logout functionality.  
*Vaine:* As easy as pie! Just use `Command` design pattern.  

### Episode 1. Command

The phrase "design pattern" woke another being.

*Niccy:* What I hear? Design Patterns?  
*Pedro:* Who are you?  
*Niccy:* I am Niccy, Angel of Dev.  
*Pedro:* Another Angel?  
*Niccy:* Correctamundo.  
*Vaine:* Don't listen to him. He always breaks someone's concentration.
Listen, first of all, you must create interface `Command` to
**encapsulate information needed to call a method at a later time**,
like in `java.lang.Runnable`.  
*Niccy:* Hahaha! Allow me to retort, Vaine, what information do you need to call a method?  
*Vaine:* Object and method name  
*Niccy:* Or just a *function* in FP world. And a way to apply it.  

Niccy showed a piece of code to Pedro.

``` clojure
(defn execute-command [command]
  (command))
```

*Niccy:* This is how abstract command pattern looks like in Clojure.
`command` is a function and `(command)` is the function call.  
*Pedro:* That simple.  
*Vaine:* But it does not support parameters for command.  
*Niccy:* Sure it does. Just wrap function with arguments to
anonymous no-arg function.  

``` clojure
(execute-command #(register "user" "password" "email"))
(execute-command #(login "user" "password"))
(execute-command #(logout "user"))
```

*Vaine:* No one understands this hash sign.  
*Niccy:* Use varargs.  

``` clojure
(defn execute-command [command & args]
  (apply command args))

(execute-command register "user" "password" "email")
(execute-command login "user" "password")
(execute-command logout "user")
```

*Niccy:* With that approach you don't even need `execute-command` function.  
*Vaine:* How to support history?  
*Niccy:* Very easy. We setting up a list to save our commands.  
*Vaine:* With concurrent modification?  
*Niccy:* Of course.  

``` clojure
(def history (atom [])) 

(defn execute-command [command & args]
  (swap! history conj [command args])
  (apply command args))
```

*Pedro:* Sorry, `history` variable is?  
*Niccy:* A vector. Every call to `execute-command` modifies `history` and
you can inspect it later. For example to report login activites.  
*Pedro:* That means...  
*Niccy:* `Command` is just a function.  

After two hours, registration, login and logout functionality was implemented
and Pedro was glad as never. The rest of the day he spent browsing the
web for funny cat image for the front page.

**Demo**

*Sven Tori:* What have you done in a week?  
*Rage Man:* We continue working on business plan for cats...and
implemented important part of our system.  
*Dale:* Correct, It's working registration, login and logout for the user.
Now you can enter your name, password and see a page with the cat.  
*Sven Tori:* Awesome! Let's checkout progress in a week. Bye all!  
*Rage Man:* Sure, bye!  
*Dale:* Bye!  
*Pedro:* Bye!  
*Sven Tori disconnected*.  
*Rage Man:* It was great.  
*Pedro:* It was a `Command` pattern, you know...  
*Rage Man:* Yeah, it was used exactly what is needed for. I had been programming before... // Grammar
Next week we are going to implement list of users. Seems reasonable?  
*Pedro:* Sure.  
*Dale:* Ok. Have a nice weekend.  

### Episode 2. Strategy

Pedro implemented admin page and table with the list of users in one day.

*Dale:* Hi, Pedro! How are you?  
*Pedro:* Fine.  
*Dale:* How do you think, what flowers Terry likes?  
*Pedro:* No idea.  
*Dale:* Ok, then, let's check what you have. Admin page, fine. List of users, fine. Emm, wait.  
*Pedro:* What's the problem?  
*Dale:* Users with subscription must come *before* other users.  
*Pedro:* There was no such requirement.  
*Dale:* But it's obvious, agreed?  
*Pedro:* Well... Yes, I'll do it.  
*Dale:* Great, good luck!  

Obviously sorting by name should work in reverse order too.
Pedro needs something that allows to
**select an algorithm's behavior at runtime**.

*Vaine:* It's candidate for `Strategy` pattern.  
*Pedro:* Yes, it seems.  
*Vaine:* Exactly it is, just like `Collections.sort(list, comparator)`  
*Pedro:* But there is no "subscription" comparator.  
*Vaine:* Write it yourself by *encapsulating needed behavior in custom comparator object*  
*Niccy:* Boom!  
*Vaine:* Again you!  
*Niccy:* Everytime I hear *encapsulate behaviour in an object* I come in.  
*Vaine:* Do not try to say I'm wrong.  
*Niccy:* You are wrong. All these *encapsulate blah blah blah* is just function.  
*Vaine:* You've already said the `Command` is just a function, now you're saying
`Strategy` is a function. Do they the same?  
*Niccy:* Kind of. `Strategy` just function accepts another function.  
*Pedro:* Enough arguing! I need to work.  
*Vaine:* Just get away, Niccy.  
*Niccy:* Hope it helps, losers!  

Nicky throw a piece with code on the table.
Pedro put it into a pocket.

*Vaine:* Listen to me. For this particular problem `sort` is an **algorithm**,
list is **data** to be sorted, and **comparator** is runtime behaviour.  
*Pedro:* Got it.  
*Vaine:* You need to follow `Comparator` interface
and provide implementation for `compare(Object o1, Object o2)` method.
Also you need another implementation `ReverseComparator`  
*Pedro:* Lot of classes...  
*Vaine:* It is flexibility, and after that you...  
*Pedro:* Got it. Got it.  
*Vaine:* Good luck!  

Pedro thought what Vaine was talking about.
All these Comparator classes seem redundant.
He took a Niccy's piece of paper from his pocket.
There were few lines of clojure.

``` clojure
(def users [{:name "felicia"     :subs true}
            {:name "faina"       :subs false}
            {:name "fernando"    :subs true}
            {:name "father Duke" :subs true}])

;; forward sort
(sort-by (juxt (complement :subs) :name) users)
;; reverse sort
(sort-by (juxt :subs :name) #(compare %2 %1) users)
```

In a next 10 minutes (*8 of them was spent to understand clojure one-liners*),
sorting functionality was done.

**Demo**

...  
*Dale:* Users with subscription appears on the top of the list.  
*Sven Tori:* Sure, because they pay us.  
*Dale:* And if we click on reverse sort it will be sorted in reverse order.  
*Sven Tori:* Awesome, they still on top.  
*Rage Man:* Guys did a good work.  
*Sven Tori:* Yes, keep it up. Good Luck!  
*Rage Man:* Bye!  
*Dale:* Bye!  
*Pedro:* Bye!  

### Episode 3. State

Monday, all-hands meeting.

*Rage Man:* We guys did a good job last week. Sven Tori allowed us to hire
marketing person. Meet **Karmen Git**, our first sales person.  
*Karmen:* Hello, all. I am Karmen and I am a sales person.  
*All:* Hi, Karmen!  
*Rage Man:* Karmen will investigate the market and keep us profitable. Questions?  
*Pedro:* What our plan for next week delivery?  
*Rage Man:* Currently, we have enough functionality to attract users, you can relax for now.  
*Pedro:* But...  
*Karmen:* You can change the cat picture on the front page, I think user needs something new.  
*Rage Man:* Excellent, Karmen. Dale, will you control that?  
*Dale:* Sure.  
*Rage Man:* Then, everybody's come back to work. Let's do the greatest cat service ever.  

Whole day Pedro was reading about usage of `Strategy` pattern. Maybe he miss something?

*Dale:* Pedro, I've sent you new picture of cat for the front page. Did you check?  
*Pedro:* One moment... Wait, what is this?  
*Dale:* Oh God, this is my torso I wanted to send to Terry. That means I sent a cat photo to her. Shit.  
*Pedro:* Happens.  

Pedro slept very bad. He had a nightmare about naked Dale with cat's head.
Disgusting.

*Karmen:* Pedro, I've investigated some users staring too much at the cat picture.
We need admin functionality to disable such users.  
*Pedro:* But, how we detect who exactly staring too much?  
*Karmen:* Unfortunately, I am not a technical guy. Talk to Dale.  

*Pedro:* Have you heard what Karmen wants?  
*Dale:* Yes, "must have" feature.  
*Pedro:* Do you think it's possible to implement?  
*Dale:* Just add functionality for admin to disable users.  
*Pedro:* And how...  
*Dale:* How, it's a Karmen's problem.  

Pedro thought about implementation.  
*Label near username must be clickable. If we click on label
user state should be changed. User can have four different states:
enabled, disabled, user with subscription and user that staring too much.
The following rules should be applied on click:
if user staring too much or enabled, we disable him; 
if user disabled we enable him; 
if user with subscription, do nothing.
Pretty straightforward. We need to make this extensible, because
I'm sure new business rules will be added later.
From code perspective we need to support some
state for...*

*Vaine:* `State`? Awesome pattern.  
*Pedro:* Do you hear my thoughts?  
*Vaine:* Yes I do.  
*Pedro:* Strange.  
*Vaine:* You definitely must use `State` pattern. It pattern helps you
**encapsulate varying behavior for the same function based on an object's state**  
*Niccy:* Hiao!  
*Vaine:* What are you need?  
*Niccy:* *"State. You're doing it wrong"*, know who said that?  
*Vaine:* Doesn't matter.  
*Niccy:* Really. State is very close to the Strategy.
They even have the same UML diagrams.  
*Vaine:* Don't start your song.  
*Niccy:* I've already started. It achieves the same goal using another mechanism,
instead of passing behavior to the method, we support some internal
state for the object, which affects method behaviour.
And from clojure perspective it can be implemented
the same way as strategy pattern. It is just a first-class function.  
*Vaine:* But successive calls can change object's state.  
*Niccy:* Right, but it has nothing to do with `Strategy` it is just implementation detail.  
*Vaine:* What about "another mechanism"?  
*Niccy:* Multimethods.  
*Pedro:* Multi *what*?  
*Vaine:* Methods, but I confused, I thought methods are in OOP.  
*Niccy:* Functions, whatever. I don't care about names.  

```
(def mr-white   (atom {:name "Mr. White"   :state :enabled}))
(def mr-pink    (atom {:name "Mr. Pink"    :state :staring}))
(def mr-blonde  (atom {:name "Mr. Blonde"  :state :subscription}))

(defmulti switch :state)

(defmethod switch :enabled [user]
  (println (:name user) "is disabled.")
  (assoc user :state :disabled))

(defmethod switch :disabled [user]
  (println (:name user) "is enabled.")
  (assoc user :state :enabled))

(defmethod switch :staring [user]
  (println (:name user) "is staring. Disable.")
  (assoc user :state :disabled))

(defmethod switch :subscription [user]
  (println (:name user) "has subscription.")
  user)

;; Usage
(swap! mr-white  switch)
(swap! mr-pink   switch)
(swap! mr-blonde switch)
```

Ton of completely new code for Pedro.
He googled keywords: *clojure multimethods*, *single dispatch*, *multimethods vs switch statement*.
Few hours of reading and he got "Aha!".
*If new state appears in requirements, we just add another defmethod, awesome!*
He used the same code Niccy gave to him, except one added line.

``` clojure
;; TODO replace println with logging
```

*Dale:* We don't have demo this week.  
*Pedro:* Great!  
*Dale:* Yes, but we plan a huge delivery next week.
By the way, enable/disable feature works great.
What did you use to implement it?  
*Pedro:* Multimethods.  
*Dale:* Multi *what*?  
*Pedro:* Methods.  
*Dale:* Ohh, got it. Methods are great. I told you are a great developer?  
*Pedro:* Yes.  
*Dale:* Remember that. Bye!  
*Pedro:* Have a nice weekend!  

### Episode 4. Template Method

*Rage Man:* Disabling users which don't want to pay
for subscription is a big step to success. But we have just one unsolved problem. Karmen?  
*Karmen:* Yeah. We have no idea how to detect such users.  
*Pedro (whispered):* You don't say.  
*Rage Man:* Let's propose some solutions and select the best one.
Does everybody familliar with brainstorming?  
*All:* Yes.  
*Rage Man:* Whatever, just to clarify.  
We generating ideas. No criticism, no slowpoking. Let's do it quick.
Ideas may be crazy, does not matter. Then we select the best one. Clear?  
*All:* Yes.  
*Rage Man:* Starting now!  
*Silence*  
*Pedro (coughing):* Khm, khm.  
*Silence*  
*Karmen (sneezing):* Aaaptsch.  
*Silence*  
*Dale (snuffling):* Thhhhhs.  
*Silence*  
*Karmen:* User could select checkbox on registration "I will never pay subscription"  
*Pedro:* Are they morons or what?  
*Rage Man:* Remember, no criticism. Writing down first idea.  
*Dale:* If user is staring at the cat more than 10 seconds,
show him an alert "Are you staring? (YES/NO)"  
*Rage Man:* Great, second idea.  
*After few hours*  
*Karmen:* For unsubscribed users we can show a picture with dog instead of cat
and if they complain, disable.  
*Rage Man:* Idea #123.  
*Dale:* Or it can be a bot, automatically inspecting all users and detect suspects.  
*Rage Man:* Idea #124. Enough. Take a lunch guys, and we will conduct another meeting in an hour.  

At the end of working day, whole team was voting and arguing about solution.
Democratic forces agreed on the bot idea, because it is
*"automatic solution do not bother customers"* (Rage Man),
*"entertain users and can be a marketable decision"* (Karmen),
*"challenging and very technical solution"* (Dale),
*"I don't know"* (Pedro).

*Rage Man:* So, currently the best idea is a bot.  
*Karmen:* Agreed.  
*Dale:* Moreover, we can implement different types of bots.
All of them will behave similarly, but on some conditions specific actions will be taken.  
*Rage Man:* Explain, please.  
*Dale:* We will implement one bot for inspecting users, another for disabling users,
yet another for saying *"Thanks!"* to users with subscription, etc.  
*Rage Man:* Seems reasonable.  
*Pedro:* Why don't we just implement one bot doing all things?  
*Dale:* It's a principle of single responsibility.  
*Pedro:* But single resp...  
*Karmen (interrupting):* We can also have a bot notifying users on their friends' birthdays.  
*Dale:* And call all of them *CatBots*.  
*Rage Man:* Ingenious. Do you see how powerful is brainstorming technique? Get back to work, guys.  

The next day Pedro was thinking how to approach the bot.
He visited a lot of gamedev forums and had no success.

*Pedro:* I have no idea how to implement that.  
*Dale:* It is simple. Few years ago I was playing MMORPG **Mech Dominore Fight Saga**.  
*Pedro:* Great! I was playing it too.  
*Dale:* Yes, the best game, in my opinion. But, there were a lot of routine tasks:
killing monsters, gathering resources, daily quests, etc.
Lot of things transformed game into a pain.  
*Pedro:* That's why I stopped playing.  
*Dale:* I did better, I implemented a bot doing all these thing automatically.  
*Pedro:* Impossible!  
*Dale:* I thought this the first time, but nothing impossible, you know.
Moreover, the default bot was easily customizable for different classes:
warriors, hunters, wizards.  
*Pedro:* What technique did you use?  
*Dale:* Don't remember exactly. It was some pattern called *"Pattern Fucntion"* or *"Pattern Method"*?  
*Pedro:* *Template Method*?  
*Dale:* Maybe. Really I don't remember. Google it.  
*Pedro:* Sure, thanks Dale.  
*Dale:* Break a leg. By the way, did you remember the cat picture I sent to Terry?  
*Pedro:* Yes.  
*Dale:* She liked it very much.
And responded with another cat image, take a look.  
*Pedro:* Funny. But if she sent you her torso it will be better ;; grammar
*Dale:* Pervert!   (все попереду)

It was the first time Dale helped. Pedro entered query in search field. Click.
Whoa! The first match.

*Pedro:* *"Template Method*, **It lets redefine certain steps of an algorithm
without changing the algorithm's structure**". Seems useful for implementing
their stupid catbot scenarios.  
*Vaine:* You just found the ideal solution for your problem.  
*Pedro:* Yes, but how to start?  
*Vaine:* I would reccomend you to inspect `java.util.AbstractList` and its abstract methods,
then write solution for Dale's bot and finally move to catbot.  
*Pedro:* Thanks, Vaine.  
*Vaine:* By the way, Niccy is not going to distract us today.
He ended up in infinite recursion with his *Functional Programming* tricks.  
*Pedro:* Did he die?  
*Vaine:* No, just stucked in "Recursion World".  
*Both laughing*  

Pedro started thinking:  
*Assume we have an algorithm to move character in some RPG world.
All characters have a lot in common: they searching for valuable items in chests,
if they found better artifact, replace their artifact, if they encountered an enemy,
attack or flee, depending on enemy strength, if they found a questgiver, accept quest...*

*But behavior between specific classes can vary depending on skills.
For example, most characters ignore closed chest, but thief can lockpick it,
most characters accept melee combat, but hunters and wizards try to keep distance...*

Phone rang.

*Pedro:* Hello?  
*Niccy:* It's Niccy.  
*Pedro:* What happened?  
*Niccy:* I was implementing modified factorial algorithm and forgot
to add exit condition. Waiting for stack overflow.  
*Pedro:* Hope it will be soon.  
*Niccy:* Definitely, few hours left. That idiot Vaine is thinking I can't help.  
*Pedro:* Did you hear our talk?  
*Niccy:* Of course, I am an angel.  
*Pedro:* Even in "Recursion World"?  
*Niccy:* Everywhere.

*Niccy sent the file tm.clj*  

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

Pedro quickly inspected the code. 

*Pedro:* But how we can override some actions based on class?  
*Niccy:* What actions? What class?  
*Pedro:* For example, thief class can lockpick the chests
or hunter class has a ranged attack.  
*Niccy:* Wait a second.  

*Niccy sent the file tm2.clj*

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

(move-to loc) ;; Warrior
(move-to loc :lock-chest lockpick) ;; Thief
(move-to loc :att-type :ranged) ;; Hunter
```

*Niccy:* You see? We just pass functions.  
*Pedro:* Unbelievable.  

Pedro used the same approach to implement *catbots*.
By the way, he added a fancy js library to visualize bots' moving.

**Demo**

*Pedro (shows the moving bots):* This bot disables users. This one is for... emmm?  
*Karmen:* Notifying users on friends' birthdays.  
*Pedro:* Right.  
*Sven Tori:* Who controls all these bots?  
*Dale:* Nobody. They are bots and that's the trick.  
*Sven Tori:* Does it mean we don't need to hire a person to control them?  
*Rage Man:* Exactly, they are fully automatic.  
*Sven Tori:* Awesome!  

### Episode 5. Iterator

*Dale:* Hi, Pedro.  
*Pedro:* Hi.
*Dale:* Can you believe, Terry invited me to her Birthday.
What can you suggest to present her?
*Pedro:* A cup.
*Dale:* A cup is банально
*Pedro:* A cup with a cat image.
*Dale:* That seems much better. Thanks.
*Pedro:* No problem.
*Dale:* By the way, I've inspected the code you using for iterating over catbots and users.
It seems kinda... unprofessional.  
*Pedro:* What do you mean?  
*Dale:* There is `for` in one place, `loop/recur` in another.  
*Pedro:* They are pretty similar.  
*Dale:* Exactly! We need to abstract iteration.  
*Pedro:* Why?  
*Dale:* To be professional.  
*Pedro:* Ohh..kay.  
*Dale:* There is `Iterator` pattern, take a look.  
*Pedro:* Sure. Thanks.  
*Dale:* Thanks to you, for the cup idea.  

Pedro started reading about iterator pattern and can't
understand its purpose. ;; TODO more

*Pedro:* Vaine, Niccy are you there?  
*Vaine:* Yes.  
*Niccy:* Ass.  
*Pedro:* Could you explain me what purpose of the iterator pattern?  
*Vaine:* Of course, it's easy.  
*Niccy:* I'm listening!  
*Vaine:* **Iterator enables a programmer to traverse a container**  
*Pedro:* Is it like `java.util.Iterator`?  
*Vaine:* Yes.  
*Pedro:* But nobody use it.  
*Vaine:* Everybody use it implicitly in `for-each` loop.  
*Niccy:* Vaine, sorry, I'm lost. What does it mean *"to traverse a container"*?  
*Vaine:* Formally, the container should provide two methods for you:  
`next()` to return next element and `hasNext()` to return true if container has more elements.  
*Niccy:* Ok. Do you know what linked list is?  
*Vaine:* Singly linked list?  
*Niccy:* Singly linked list.  
*Vaine:* Sure. It is a container consists of nodes. 
Each node has a data value and reference to the next node.  
*Pedro:* And `null` value if there is no next node?  
*Niccy:* Correct. Now tell me how traversing such list is differs
from traversing via iterator?  
*Vaine:* Emmm...  

Vaine wrote two traversing snippets:

* Traversing using iterator

``` java
Iterator i;
while (i.hasNext()) {
  i.next();
}
```

* Traversing using linked list

``` java
Node next = root;
while (next != null) {
  next = next.next;
}
```

*Vaine:* They are pretty similar...  
*Pedro:* I think so. What is analogue of `Iterator` in clojure?  
*Niccy:* `seq` function.  
*Vaine:* Why not list?  
*Niccy:* Why not? Guess what returns applying `seq` to a container?  
*Pedro:* A list?  
*Niccy:* Ingeniously.

``` clojure
(seq [1 2 3])       => (1 2 3)
(seq (list 4 5 6))  => (4 5 6)
(seq #{7 8 9})      => (7 8 9)
(seq (int-array 3)) => (0 0 0)
(seq "abc")         => (\a \b \c)
```

*Vaine:* Ok, I agree. But they just called java iterator with another name.  
*Niccy:* Actually, it's java called list an *"iterator"*.  
*Vaine:* ...  
*Niccy:* And, in fact, `seq` is much better than iterator.  
*Vaine:* Why?  
*Niccy:* It is immutable and persistent.
That means you won't have concurrency problems
with `seq` at all.  
*Vaine:* Makes sense, but this function works only on
clojure data structures, what if I want implement custom one?  
*Niccy:* You do. Using `deftype` you can create new classes the same way as in java.
To make `seq` work on new class, just implement `seq` function
from `clojure.lang.Seqable` interface.  

``` clojure
(deftype ReverseArray [vec]
  clojure.lang.Seqable
  (seq [self] 
    (seq (reverse vec))))

(def ra (ReverseArray. [1 2 3]))
(seq ra)   =>  (3 2 1)
```

*Vaine:* Ha! It's just like implementing an `Iterator`.  
*Niccy:* I suppose it is a plus.  
*Vaine:* Ok, you win the battle, but didn't win a war.  
*Niccy:* War consists of battles. If I win every battle...  
*Vaine:* Stupid!  
*Vaine disappeared*  
*Pedro:* Niccy and how `seq` is applied to...  
*Niccy:* `(doc seq)`  
*Niccy disappeared*  

;; Pedro wrote something
;; Dale come and said everything is good

### Episode 6. Visitor

Bot idea was very attractive from Sven's perspective.
*"More bots!"*, he said.

...  
*Rage Man:* Guys, what we planned for this week?  
*Karmen:* I investigated the market. We can expand our services  
by introducing referral program.  
*Rage Man:* Explain please.  
*Karmen:* Existing users can invite other users for some benefits.  
*Rage Man:* What benefits?  
*Karmen:* Temporary subscription or cat-style avatar.
I still experimenting with this.  
*Dale:* And if invited users will invite other users we'll grow exponentially.  
*Pedro:* 6 steps and we dominate!  
*Rage Man:* Ok, what about bots?  
*Dale:* What bots?  
*Rage Man:* Sven just said *"more bots"*.  
*Karmen:* We'll think about it.  

There was no problem to implement referral system. Pedro handled it very easily.
Just added database column `referrer` for user table to indicate by whom this user was invited.
Magical `NULL` was used to indicate users with no-referral registration.

*Incoming call from karmelove81*
*Karmen:* Hi, Pedro.
*Pedro:* Hello, Karm.
*Karmen:* We decided to implement *"PayBot"*
*Pedro:* Pay?
*Karmen:* Yes, it will give some bonus points to each 
user based on some conditions. I sent you an email with rules.
*Pedro:* But... We don't have bonus points in user account.
*Karmen:* Really? Let me confirm.
*karmelove81 disconnected*

Pedro received email from Karmen and clicked 
to open attachment `business_rules.jpg`

*Incoming call from karmelove81*
*Karmen:* Hi, Pedro. I confirmed. We really don't have bonus points yet.
*Pedro:* As I said.
*Karmen:* That means we must implement them.
*Pedro:* It's not planned delivery.
*Karmen:* I'll confirm with Rage Man, but start working.
*karmelove81 disconnected*

*Pedro:* Asylum.

Pedro added another column `bonus_points` to user table. Done.
He went to drink some coffee, but some strange feeling was as
he forgot something.

*Pedro:* Attachment!

Attachment `business_rules.jpg` was just a photo
of terribly-written text on the piece of paper.

* If user is no-referral registered, with subscription +100 points.
* If user is no-referral registered, enabled +50 points.
* If user is no-referral registered, disabled +10 point.
* If user is referral registered, with subscription +50 points.
* If user is referral registered, enabled +10 point.
* If user is referral registered, disabled 0 point.
* Also, for each user add points per invited user
  * 5 for no-referral
  * 3 for referral
  
*Pedro:* She's, probably, never heard of cross product.

Pedro started to implement the rules directly via if/else conditions.
Very soon he was lost in his own code.

*Dale:* How thigs are going?  
*Pedro:* Not so good, thinking about this PayBot.  
*Dale:* What's the problem?  
*Pedro:* There are a lot conditions to check during visiting user in iteration.
I just don't want if/else mess.  
*Dale:* If bot should visit user, it's definitely candidate for *visitor pattern*.  
*Pedro (trying to joke):* V for Visitor.  
*Dale (laughing):* Ahaha! I can use this joke to laugh with Terry. How do you think, will she understand?  
*Pedro:* Doubt. Just open the dictionary and find another word starting with "V".  
*Dale:* Good idea!  

Pedro was trying to guess what horny word Dale 
will choose to impress Terry.

*Pedro:* Stop talking! I need to concentrate on the problem.  

Search query *"visitor pattern"* didn't clear his mind.
Every source was explaining something completely different.

*Pedro:* I really can't understand what is Visitor pattern for.  
*Vaine:* Yeah, it is probably the most complicated pattern.  
*Pedro:* Wiki says it is a **way of separating an algorithm
from an object structure on which it operates**.  
*Vaine:* Kind of. I'll give you an example.
Assume you have a different shapes: rectangle, triangle, circle, etc.  
*Pedro:* Yes.  
*Vaine:* How would you implement them.  
*Pedro:* I can create an abstract `Shape` class and all
other shapes will extend it.  
*Vaine:* Why would you do that?  
*Pedro:* It's because parent `Shape` could contain some method
common for all shape types, but differs in implementation.  
*Vaine:* Like what?  
*Pedro:* Like square function.  
*Vaine:* Great. It's easy to add new type to hierarchy.
Now, assume you want to render a shape.  
*Pedro:* I will add `render` method to `Shape` class.  
*Vaine:* And implement shape-specific rendering in all shapes?  
*Pedro:* ..Yes..  
*Vaine:* Ok, now you got another bunch of methods. You need to
modify all your one million (*how much?*) shapes.  
*Pedro:* Compiler is fast.  
*Vaine (laughing):* Hahaha! It is not the way professional
programmers think. This problem is called **expression problem**.  
*Pedro:* I've never heard about it before.  
*Vaine:* In OOP languages easy to add new types,
but hard to add new operations on them.
Visitor pattern solves this problem.  
*Pedro:* And what about FP languages?  
*Vaine:* Hush!  
*Pedro:* ...  
*Vaine (whisper):* I don't want a Niccy to hear us.
FP languages have reverse problem.
Easy to add new operations and hard to add new types.  
*Pedro:* And what pattern to use in that case?  
*Vaine:* Really, I don't know. FP is something strange for me.  
*Niccy appeared with colander on the head*  
*Niccy:* Nothing strange. It is clear problem that can be 
solved by **double dispatch**.  
*Vaine:* Not all languages support double dispatch.  
*Niccy (laughing):* That's what I say every time.  
*Pedro:* I am aware of *single dispatch*, but...  
*Niccy:* If you have a call `object.method(argument)`, you see
**runtime polymorphism** in action. Actual `method` will be selected
depending on the type of `object`.  
*Pedro:* It's obvious.  
*Niccy:* It's obvious fail. Runtime can't use 
information of `argument` type to select an appropriate method.  
*Vaine:* Sure runtime is aware about `argument` type.  
*Niccy:* You didn't get. Just look at the code:  

``` java
class Food { }
class FastFood extends Food { }

class Coder {
	void eat(Food food) { 
		System.out.println("Programmer eats Food");
	}
	void eat(FastFood food) {
		System.out.println("Programmer eats FastFood");
	}
}

public static void main(String[] args) {
	Food     f1 = new Food();
	FastFood f2 = new FastFood();
	Food     f3 = new FastFood();
	
	Coder c = new Coder();
	c.eat(f1); // Coder eats Food
	c.eat(f2); // Coder eats FastFood
	c.eat(f3); // Coder eats Food
}
```

*Vaine:* Hmm... Why would you need to dispatch on `argument` type?  
*Niccy:* You don't need to.  
*Vaine:* Hey, really, explain.  
*Niccy:* Who was talking about *expression problem*?
*Vaine:* ...
*Pedro:* Seems we can solve this problem by using argument
as an object and pass object as an argument in addition to first call.  
*Niccy:* This is exactly a hack Visitor uses.  
*Vaine:* Is there any better alternative?  
*Niccy:* Multimethods.  
*Both Vaine and Pedro:* Multi *what*?  
*Niccy:* Multi *youaremorons* and adhoc hierarchies.  
*Vaine:* Proofcode!  
*Niccy:* Ha! Easy:  

``` clojure
(derive ::fastfood ::food)

(defmulti eat (fn [a b] [a b]))

(defmethod eat [::coder ::food] [x y]
  (println "Coder eats Food"))
  
(defmethod eat [::coder ::fastfood] [x y]
  (println "Coder eats FastFood"))

(eat ::coder ::food)     => Coder eats Food
(eat ::coder ::fastfood) => Coder eats FastFood

;; if [::coder ::fastfood] is missing
;; then [::coder ::food] is used for dispatch
```

Ten minutes staring to the code in silent room.

*Pedro:* I'm frustrated.  
*Vaine:* **Quadrocolumn**! That's the face of FP monster, I'm done with that.  
*Vaine is leaving*  
*Niccy:* Hahaha! What the hard part?  
*Pedro:* What is *quadrocolumn*?  
*Niccy:* Namespace qualified keywords. Read about it.  
*Pedro:* Emm...   
*Niccy:* You want to ask how is it better? I'll tell you.
`derive` can be used to create adhoc hierarchies, even using multiple inheritance,
`parents/ancestors/descendants` methods are used to retrieve hierarchy structure,
hardly achievable in OOP and `prefer-method` can be used if there is any ambiguity.  
*Pedro:* ...  
*Niccy:* I'm sure you'll understand.  

Pedro was experimenting with multimethods,
adhoc hierarchies and practicing multiple dispatch
until he was enlightened.

*Pedro:* As easy as pie!

``` clojure
(def user {:name "William"
           :state :subscription
		   :referral false
		   :bonus 0
		   :referrers [user1 user2 user3]})
;; TODO good example
```

**Demo**

...
*Sven Tori:* I'm very impressed, guys. 
You've implemented bonus points functionality 
in such limited period. It's awesome!
*Rage Man:* Thanks!
*Karmen:* Sven, we have several feature requests from users.
*Dale:* Because of feedback system we integrated.

### Episode 7: Memento

**No purpose at all**
We just saving this to atom/ref

*Dale* *Tells joke about V for *


### Episode 8: Mediator

???

### Episode 9: Observer

New thanks person

### Episode 10: Chain of responsibility

Logging?

### Episode 11: Interpretator

// bencode?
Our users opened torrent tracker with our cat videos
need to decode B-encode format


## TODO

---

## Season II. Creational

Seed Investments, Startup summit


### Episode 12: Prototype

"type of objects to create is determined by a prototypical instance,
which is cloned to produce new objects"

*Example:* `Object.clone()`

In mutable world we need `clone()` operation to create
new object with the same properties.
The hard part is the copy must be deep, i.e. instead of copying
reference you need recursively `clone()` object.

Clojure creates new copy when you modifyng object.

Imagine group of students in the class.
All of them share a lot of properties: school, class, age.
Only names are different.

We start from creating prototype object:

``` clojure
(def student-prototype
  {:name "TODO"
   :age 12
   :school "School #13"
   :class "5-B"})
```

To create a specific person you just modify a prototype

```
(def create-student [name]
  (assoc student-prototype :name name))
```

Modifying a map is a very effective operation,
it does not need to copy all values, just changed ones. Magic.

If you have a string collection with `names`,
to create student objects based on these names
as simple as never.

``` clojure
(def students (mapv create-student names))
```

*Note:* `mapv` produce vector instead of list.

Suddenly, third student celebrates its birthday.
We must increment his age.

``` clojure
(update-in students [2 :age] inc)
```

School got investments from Google and decided to rename it as "Schoogle #13"

``` clojure
(map #(assoc % :school "Schoogle #13") students)
```

Simple and clear.

**Conclusion:** Prototype is not needed in immutable world.

## Singleton (Simpleton)

"restricts the instantiation of a class to one object"

*Example:* `java.lang.Runtime`

Believe or not, it's just a fancy global variable.

`(def singleton [:data])`

Oh, sorry it must be mutable.

`(def singleton (atom [:data]))`








### Roles

With the lack of imagination all characters
and names are anagrams of some words.

**Pedro Veel** - Developer  
**Serpent Hill & R.E.E.** - Enterprise Hell  
**Weats Inc.** - Waste Inc.  
**Sven Tori** - Investor  
**Mate Dale** - Team Lead  
**Rage Man** - Manager  
**Terry P** - Pretty  
**Vaine** - Naive  
**Niccy** - Cynic  
**Karmen Git** - Marketing  
**Mech Dominore Fight Saga** - Heroes of Might and Magic  
