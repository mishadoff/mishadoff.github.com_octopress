---
layout: post
title: "Clojure Design Patterns"
date: 2014-05-26 00:09
comments: true
categories: [clojure, programming, java, story, patterns]
published: false
---

*Quick* overview of the classic [Design Patterns](http://en.wikipedia.org/wiki/Design_Patterns) in Clojure

<!-- more -->

**Disclaimer:** *Most patterns are easy to implement because
we use dynamic typing, functional programming and, of course,
Clojure. Some of them look wrong and ugly. It's okay.
All characters are fake, coincidences are accidental.*

> Our programming language is fucked up.
> That's why we need design patterns.
> -- Anonymous

### Intro

Two modest programmers **Pedro Veel** and **Eve Dopler**
solving 23 software engineering problems and
applying design patterns.

### Episode 1. Command

> Leading IT service provider **"Serpent Hill & R.E.E"**
> acquired new project for USA customer.
> First delivery is a register, login and logout functionality.

*Pedro:* Oh, that's easy. You just need a Command interface...  

``` java
interface Command {
  void execute();
}
```

*Pedro:* Every action should implement it and define `execute` behaviour.

``` java
public class LoginCommand implements Command {

  private String user;
  private String password;

  public LoginCommand(String user, String password) {
    this.user = user;
    this.password = password;
  }

  @Override
  public void execute() {
    DB.login(user, password);
  }
}
```

``` java
public class LogoutCommand implements Command {

  private String user;

  public LogoutCommand(String user) {
    this.user = user;
  }

  @Override
  public void execute() {
    DB.logout(user);
  }
}
```

*Pedro:* Usage is simple as well.

``` java
(new LoginCommand("Johnny", "qwerty")).execute();
(new LogoutCommand("Johnny")).execute();
```

*Pedro:* What do you think, Eve?  
*Eve:* Long enough. Look at this  

``` clojure
(defn execute [command]
  (command))

(execute #(db/login "Johnny" "qwerty"))
(execute #(db/logout "Johnny"))
```

*Pedro:* What the hell this hash sign?
*Eve:* Just a shortcut for javaish  

``` java
new SomeInterfaceWithOneMethod() {
  @Override
  public void execute() {
    // do
  }
};
```

*Pedro:* Hmm...  
*Eve:* Or if you want - no-hash solution.  

``` clojure
(defn execute [command & args]
  (apply command args))

(execute db/login "Johnny" "qwerty")
```

IMPROVE

*Pedro:* Why you just don't call `(db/login "Johnny" "qwerty")`?  
*Eve:* Bingo!  
*Pedro:* What do you mean?  
*Eve:* **Command is just a function.**

### Episode 2. Strategy

> **Sven Tori** pays a lot of money and
> want to see a page with list of users.
> But users must be sorted by name and
> users with subscription must appear *before*
> all other users.
> Obviously, because they pay.
> And obviously reverse sorting should keep subscripted users on top.

*Pedro:* Ha, just call `Collections.sort(users, comparator)`
with custom comparator. `comparator` is a strategy.   
*Eve:* How would you implement it?  
*Pedro:* You need to take `Comparator` interface
and provide implementation for `compare(Object o1, Object o2)` method.
Also you need another implementation for `ReverseComparator`  
*Eve:* Stop talking, show me the code!  

``` java
class SubsComparator implements Comparator<User> {

  @Override
  public int compare(User u1, User u2) {
    if (u1.isSubscription() == u2.isSubscription()) {
      return u1.getName().compareTo(u2.getName());
    } else if (u1.isSubscription()) {
      return -1;
    } else {
      return 1;
    }
  }
}

class ReverseSubsComparator implements Comparator<User> {

  @Override
  public int compare(User u1, User u2) {
    if (u1.isSubscription() == u2.isSubscription()) {
      return u2.getName().compareTo(u1.getName());
    } else if (u1.isSubscription()) {
      return -1;
    } else {
      return 1;
    }
  }
}

// forward sort
Collections.sort(users, new SubsComparator());

// reverse sort
Collections.sort(users, new ReverseSubsComparator());
```

*Pedro:* Could you do the same?  
*Eve:* Yeah, something like that  

``` clojure
(sort (comparator 
       (fn [u1 u2]
         (cond
          (= (:subscription u1)
             (:subscription u2)) (neg? (compare (:name u1)
                                                (:name u2)))
             (:subscription u1) true
             :else false))) users)
```

*Pedro:* Pretty similar.  
*Eve:* But we can do it better  

``` clojure
;; forward sort
(sort-by (juxt (complement :subscription) :name) users)

;; reverse sort
(sort-by (juxt :subscription :name) #(compare %2 %1) users)
```

*Pedro:* Oh my gut! Monstrous oneliners.  
*Eve:* Functions, you know.  
*Pedro:* Whatever, it's very hard to understand what's happening there.  
*Eve explains juxt, complement and sort-by*  
*10 minutes later*  
*Pedro:* Very doubtful approach to pass strategy.  
*Eve:* **Strategy is just a function accepts another function.**  

### Episode 3. State

> Sales person **Karmen Git**
> investigated the market and decided
> to give different type of users different functionality.

*Pedro:* Smooth requirements.  
*Eve:* Let's clarify them.  

- *If user has subscription show him all news in a feed*
- *Otherwise, show him top 10 news*
- *If he pays money, they are added to his account balance*
- *If user doesn't have subscription and there is enough money
to buy subscription, change his state to...*

*Pedro:* State! Awesome pattern. First we make a user state enum  

``` java
public enum UserState {
  SUBSCRIPTION(Integer.MAX_VALUE),
  NO_SUBSCRIPTION(10);

  private int newsLimit;

  UserState(int newsLimit) {
    this.newsLimit = newsLimit;
  }

  public int getNewsLimit() {
    return newsLimit;
  }
}
```

*Pedro* User logic is following  

``` java
public class User {
  private int money = 0;
  private UserState state = UserState.NO_SUBSCRIPTION;
  private final static int SUBSCRIPTION_COST = 30;

  public List<News> newsFeed() {
    return DB.getNews(state.getNewsLimit());
  }

  public void pay(int money) {
    this.money += money;
    if (state == UserState.NO_SUBSCRIPTION
	    && this.money >= SUBSCRIPTION_COST) {
      // buy subscription
      state = UserState.SUBSCRIPTION;
      this.money -= SUBSCRIPTION_COST;
    }
  }
}
```

*Pedro:* Lets call it  

``` java
User user = new User(); // create default user
user.newsFeed(); // show him top 10 news
user.pay(10); // balance changed, not enough for subs
user.newsFeed(); // still top 10
user.pay(25); // balance enough to apply subscription
user.newsFeed(); // show him all news
```

*Eve:* You just hide value that affects  
behaviour inside `User` object. We could use strategy to pass it directly `user.newsFeed(subscriptionType)`.  
*Pedro:* Maybe, State is very close to the Strategy.  
They even have the same UML diagrams. but we encapsulate balance and bind it to user.
*Eve:* I think it achieves the same goal using another mechanism,
from clojure perspective it can be implemented
the same way as strategy pattern. **It is just a first-class function**.  
*Pedro:* But successive calls can change object's state.  
*Eve:* Correct, but it has nothing to do with `Strategy` it is just implementation detail.  
*Pedro:* What about "another mechanism"?  
*Eve:* Multimethods.  
*Pedro:* Multi *what*?  
*Eve:* Look at this  

``` clojure
(defmulti news-feed :user-state)

(defmethod news-feed :subscription [user]
  (db/news-feed))

(defmethod news-feed :no-subscription [user]
  (take 10 (db/news-feed)))
```

*Eve:* And `pay` function it's just plain a function change state of object. We don't like state too much in clojure, but if you wish.  

``` clojure
(def user (atom {:name "Jimmy"
                 :balance 0
                 :user-state :no-subscription}))
				 
(def ^:const SUBSCRIPTION_COST 30)

(defn pay [user amount]
  (swap! user update-in [:balance] + amount)
  (when (and (>= (:balance @user) SUBSCRIPTION_COST)
             (= :no-subscription (:user-state @user)))
    (swap! user assoc :user-state :subscription)
    (swap! user update-in [:balance] - SUBSCRIPTION_COST)))

(news-feed @user) ;; top 10
(pay user 10)
(news-feed @user) ;; top 10
(pay user 25)
(news-feed @user) ;; all news
```

*Pedro:* Is dispatching by multimethods better than dispatching by enum?  
*Eve:* No this particlular case, but in general yes.  
*Pedro:* Explain, please  
*Eve:* Do you know what *multiple dispatch* is?  
*Pedro:* Not sure.  
*Eve:* Well, then we need to discuss `Visitor` pattern.

### Episode 4. Visitor



### Episode 4. Template Method

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
		   :registration ::base
		   :bonus 0
		   :ref-num 12})

(derive ::referral ::base)


```

**Demo**

...
*Sven Tori:* I'm very impressed, guys. 
You've implemented bonus points functionality 
in such limited period. It's awesome!
*Rage Man:* Thanks!
*Karmen:* We plan to build a store-system for users. They can
buy some virtual items for their bonus points.
*Sven Tori:* I see here a huge place for monetisation.
*Karmen:* Exactly. By the way, we have several feature requests from users.
*Dale (interrupting):* Because of feedback system we integrated.

### Episode 7: Memento

*Rage Man (in panic):* Pedro, what's going on?
*Pedro:* What's up?
*Rage Man:* There is a user in the system with bonus points over one million.
*Pedro:* I..I... don't know.
*Rage Man:* Nothing works.
*Pedro:* Everything works, it's just some bug.
*Rage Man:* It's not *"just bug"*, it's our money.
*Pedro:* I understand, I'll investigate.
*Rage Man:* We need fix for that as soon as possible.

Pedro wasn't good in security bu he used as much prevention mechanism as. 
;; more

*Pedro:* Impossible. There is only one place which 
modifies user bonus points - it is a PayBot.
*Pedro:* There is no problems with PayBot controling. 
It still not managed by admin, fully server code and running by timer.

Pedro took a look at the referers code.

*Pedro:* Oh, shi~

Pedro incremented `ref-num` variable right after user sent invite.
Some user sent invites to million of fake accounts. They didn't register.

*Dale:* Hi, Pedro. Did you hear about user with 1M+ bonus points?
*Pedro:* Yes, I've found the problem and aalready fixed it.
*Dale:* It is great! What was the root cause?
*Pedro:* It's my fault.
*Dale:* Everybody make mistakes. Don't embarass yourself. Really, what was the prbolem?
*Pedro:* I was incrementing referal-registered users after user sent invites. 
Not when invite was approved.  
*Dale:* Understand. Everybody make mistakes. 
However, you are good developer.
*Pedro:* Thanks...

**Unplanned meeting**

*Rage Man:* User with ONE MILLION BONUS POINTS!!!
*Dale:* Keep calm, everybody make mistakes.
*Pedro:* As I said it is already fixed.
*Rage Man:* Fixed what?
*Pedro:* The problem with bonus points. It was caused by invalid...
*Rage Man:* I don't care about technical things.
The user with ONE MILLION BONUS POINTS still in the system!
*Dale:* Oh my god! We forgot to nullify his points.
*Rage Man:* That's the problem! And there is probably other users
with fake points, how to detect them?
*Dale:* ...
*Pedro:* ...
*Rage Man:* I want solution tomorrow.

Meeting has ended. blah blah

*Pedro:* We are fucked up.
*Dale:* Remember the joke *"V for Visitor"*?
*Pedro:* Yes.
*Dale:* Actually, now is *"V for Vaseline"*.
*Both laughing*
*Pedro:* Кстати, did you tell Terry joke about Visitor?
*Dale:* No.
*Pedro:* And what did you tell her?
*Dale:* Does not matter.
*Pedro:* Hey, I'm really interesed.
*Dale:* *"V for Vagina"*
*Pedro:* What a pervert!
*Karmen:* Sorry, guys, for interruping.
I think I've gound a solution.
*Dale:* We are listening.
*Karmen:* What about periodically save user bonus points. 
*Pedro:* Like shapshot?
*Karmen:* What screenshot?
*Pedro:* Nevermind.
*Karmen:* And compare it's last state with current, if there is big deviation
we will investigate the issue.
*Dale:* And will be able to revert it's points!
*Karmen:* This also will prevent some issues in future.
*Dale:* Karm, you are genious! Pedro, do you see some issues with that.
*Pedro:* No.
*Dale:* Great! Will be done tomorrow.
*Karmen:* Thanks, guys.
*Dale and Pedro:* Thanks, Karm!

Pedro started working. He thought some of previous patterns 
could support this functionality. State? Command?

*Vaine:* Memento.
*Pedro:* Www...what?
*Vaine:* Memento pattern.
*Pedro:* Oh, i thought you mean movie.
*Niccy:* Good movie. Definitely better than the pattern.
*Vaine:* Niccy, could you don't interrupt me.
*Niccy (sarcastic):* Sure. I mean it is the world greatest pattern ever.
*Vaine:* Ok. So the Memento **provides the ability to restore an object to its previous state**
*Pedro:* Got it.
*Niccy:* Got it.
*Vaine:* To implement it you need three objects: *originator*, *caretaker*
and a *memento*
*Niccy (whisper):* Or two functions: `save` and `restore`
*Vaine:* *The originator is...*
*Pedro and Niccy talking during Vaine's monologue*
*Pedro:* How two functions handle this?
*Niccy:* One is saving state, another is restoring.
*Vaine:* *...the internal state of an object...*
*Pedro:* Where is state?
*Niccy:* Atom is good.
*Vaine:* *...The caretaker is going...*
*Pedro:* So, you mean if I have some user, I can just save needed values to an atom?
*Niccy:* Exactly.
*Vaine:* *...able to restore previous state...*

``` clojure
(def user (atom {:name "Memento" :points 10 :state :disabled :referer nil}))
(def memento (atom {}))

(defn save []
  (swap! memento @user))

(defn restore []
  (swap! user @memento))
```

*Pedro:* And if there are lot of users?
*Niccy:* `map`
*Vaine:* *...a memento object...*
*Pedro:* An atom per user?
*Niccy:* If you want to perform `restore` per user, then yes.
*Vaine:* *...to the originator...*
*Pedro:* Understand. Now it is very easy to implement what they want.
*Niccy:* Take care.
*Vaine:* *...caretaker.* Did you understand?
*Pedro:* Sure.
*Vaine:* Where is Niccy?
*Pedro:* He is gone. Probably was bored by your mono... uhh.. talk.
*Vaine:* Haha, loser! Goodbye, Pedro.
*Pedro:* Bye, Vaine. And.. thanks for explanation.
*Vaine:* No problem.

Pedro was thinking 

*Pedro:* Why these *"two functions"* were called *pattern*?
*Pedro:* Did I miss something?
*Pedro:* Anyway, all works as expected.

The rest of the day Pedro included js library for charts to
visualize bonus points deviation betwen states.

**Demo**

*Sven Tori:* I heard some user had one million bonus points?
*Rage Man (lie):* It is hacker. He broke our system and increased his points.
*Sven Tori:* It is very bad, we don't want such situations in future.
*Rage Man:* Ah.. Sure. We implemented a system allows us to monitor intruders.
*Dale shows the system*
*Sven Tori:* What a..an AWESOME charts! Very exciting work.
*Rage Man:* We are just professionals.
*Sven Tori:* I know. Good work guys, as always. Bye!
*Sven Tori disconnected.*

The rest of the meeting Rage Man was explaining to the team,
why he was lying to Sven. End of the week.

### Episode 8: Mediator

*Dale:* Hi, Pedro. How are you?
*Pedro:* Fine. And you?
*Dale:* Me and Terry were at the CAS.
*Pedro:* CAS? Compare and Swap?
*Dale:* Contemporary Art Show.
*Pedro:* Ou. And what?
*Dale:* Strange Art. Tomato soup preserve floating in the piano.
*Pedro:* WTF?
*Dale:* I said, it's strange. But Terry liked.
*Pedro:* Be careful, it can be a sign.
*Dale:* She is a very creative person, instead of us...
*Pedro:* We are just slaves.
*Dale:* *"We are just professional"*
*Both laughing*
*Dale:* Pedro, everyone are really excited about your visualisations.
*Pedro:* I'm glad. Everything was done in library.
*Dale:* What library?
*Pedro:* `JCharts`
*Dale:* Ok, I will take a look.

Few minutes passed.

*Dale:* Pedro!
*Pedro:* Huh?
*Dale:* JCharts is forbidden for commercial usage.
*Pedro:* What you mean forbidden?  
*Dale:* We should pay for it.
*Pedro:* What's the problem? We've got investor, he has money and likes charts.
*Dale:* I mean, everybody loves charts unless they are paid.
*Pedro:* Ok, I will remove that library.
*Dale:* No, stop. Are there any similar libraries?
*Pedro:* Maybe, need investigation.
*Dale:* Here is the deal: refactor the code to abstract its usage.
*Pedro:* I didn't understand.
*Dale:* We should have some unified API for charts.
Like `PieChart(int[] volumes, String[] labels)`
*Pedro:* Why we need that?
*Dale:* It helps to avoid high coupling. We can easily change the charts vendor,
implement the connection, but code for charts usage will remain the same.
*Pedro:* Got it.
*Dale:* And I will try to reach agreement about `JCharts` usage.
If Sven gives money, we are good. If not, we just change charts vendor.
*Pedro:* Seems reasonable.

Another refactor task. Unified API and vendors. Pretty much like
JPA and Hibernate. Hibernate. Hi, bear night. Pedro grimaced. It reminded him
his job few years ago.

Daydreaming about monster trying to eat little girl toby
*Reference to reservoir dogs

*Vaine:* Don't sleep!
*Pedro:* Wh..Wha..What happened? Where is Toby?
*Vaine:* There is no Toby. You just dreamt.
*Pedro:* Oh, thanks god...
*Vaine:* You must work.
*Pedro:* I know, I just... Stop! How did you know about dream?
*Vaine:* I am an Angel.
*Pedro:* I am a bit bored of this magic things.
*Vaine:* Don't you want help? Then I'm leaving.
*Pedro:* Wait, sss..sorry. I still under nightmare effect.
*Vaine:* I know.
*Pedro:* ...
*Vaine:* Just kidding. Let's solve your refactor problem.
*Pedro:* Ok.
*Vaine:* You need low coupling on the one side
and components interactions on the other.
*Pedro:* Just coupling, no interactions.
*Vaine:* Are you sure? If you have different views
for the same set of widgets, can be they somehow connected?
*Pedro:* Maybe.
*Vaine:* It is interaction.
*Pedro:* Ok. Low coupling and interactions.
*Vaine:* Let me think.
*Niccy:* ...And the first place goes to the Mediator!
*Vaine:* Oh, Niccy, thanks. Definitely Mediator.
It **defines an object that encapsulates how a set of objects interact**
*Pedro:* It is structural?
*Vaine:* No. Interaction behaviour may change at a runtime. 
*Niccy:* Example, please.
*Vaine:* No problem. Pedro, what do you like?
*Pedro:* Like what?
*Vaine:* Like whatever.
*Pedro:* I don't know...Cars?
*Vaine:* Cars? Excellent. Car is a mediator.
*Niccy:* Expected.
*Vaine:* Car consists of several parts: wheels, engine, doors,
bumper, lights, accumulator, steering, etc.
Every part assumed to function independently of other.
*Pedro:* Correct.
*Vaine:* But its impossible. If steering rotates, wheels are also rotate.
If engine is broken, the car is not moving.
*Pedro:* Then steering should have references to wheels and if...
*Vaine:* No. The whole mediator pattern idea is to make steering don't know
anything about wheels.
*Pedro:* Interesting. How do achieve this in code?

``` java
class Car {
  Wheel[] wheels;
  Engine engine;
  Steering steering;

  // rotate steering
  void rotate(int angle) {
    steering.rotate(angle);
	// move front wheels
	wheel[0].rotate(angle);
	wheel[1].rotate(angle);
  }
}
```

*Vaine:* Code is pretty obvious.
*Pedro:* Great! We are just controlling `Car` object?
*Vaine:* Yes.
*Niccy:* It is not mediator. You are just hiding Car parts functionality
behind Car.
*Vaine:* Um... Ok. Take this method.

``` java
double gasoline;
double gasConsumptionPerMeter;

// move car forward by meters
void move(double meters) {
  double consumption = gasConsumptionPerMeter * meters;
  if (!engine.isBroken() && (consumption < gasoline)) {
    for (Wheel w : wheels) {
      w.move(meters);
    }
	gasoline -= consumption;
  }
}
```

*Vaine:* It depends on engine and gasoline volume.
*Niccy:* Much better. But I don't understand where is the pattern?
*Vaine:* Open your eyes.
*Niccy:* No, really. It's the code everyone can write without 
reference to "pattern" word.
*Vaine:* Write.

``` clojure
;; gasoline is an atom
(defn move-car [meters]
  (let [consumption (* gasConsumptionPerMeter meters)]
    (if (and (broken? engine) (< consumption @gasoline))
	  (map move-wheel w))
	(swap! gasoline - consumption)))
```

*Vaine:* Exact copy.
*Niccy:* Yes. Just usual code. Where is pattern?
*Vaine:* Pattern is...
*Niccy:* I explain. Imagine IT company.
Customer says to Product owner "I want feature to be done".
Product owner estimates a budget and resources for the feature
and checks if company has available resource.
He opens request for resources in some internal resource system.
Someone has approved request. He doesn't know who. He doesn't care who.
Next step. Product owner has resources and opens feature request.
Project Manager sees feature request, "Oh, it means we have resources".
He splits feature to smaller tasks and put all tasks to some internal
tasks system. Developer gets notification "New tasks available".
He take this task, do it. And resolves. QA sees the list of resolved tasks
and verifies each one. As soon as all feature subtasks resolved and verified.
Project Manager resolves feature request. Product owner sees resolved request
and email to customer "Done.". That's how things works.
*Pedro:* Actually this is not true. We have a lot of meetings where we interact.
*Niccy:* Meetings corrupt Mediators.
*Everybody is laughing*
*Pedro:* But I finally understand what is mediator.
*Vaine:* Glad to be useful.
*Niccy:* Ness.
*Both disappeared*
*Pedro:* Wait! How can I apply this pattern to my problem?..

*Incoming call from cheapanddale*
*Dale:* Pedro?
*Pedro:* Yes.
*Dale:* I asked about using `JCharts`. 
I have two items of news. Good and bad.
*Pedro:* Good?
*Dale:* We allowed to use `JCharts`. And no additional work required.
*Pedro:* It's fantastic. What is bad?
*Dale:* We are buying subscription only for one months so we need another
library and additional work is required.
*Pedro:* ...
*Dale:* But, I'm sure you'll handle it. Bye.
*cheapanddale disconnected*
*Pedro:* At least I won't die of boredom.

Pedro started inspecting the old charts-related code to detect
candidates for Mediator. This one is definitely for refactoring.

**Before**

``` clojure
(defn pie-chart [data header-panel refresh-panel
                      notification-panel content-panel]
  (set-text header-panel "Pie Chart")
  (set-font header-panel :H1 :bold)
  (set-color header-panel Color/BLACK)
  (set-text refresh-panel (date-format "YYYY-MM-dd HH:mm" (Date.)))
  (set-color refresh-panel Color/BLACK)
  (set-content content-panel (render-chart :pie data))
  (set-color notification-panel Color/GREEN)
  (set-font notification-panel :H3 :italic :bold)
  (set-text notification-panel "Chart is completed"))

(defn bar-chart [data header-panel refresh-panel
                      notification-panel content-panel]
  (set-text header-panel "Bar Chart")
  (set-font header-panel :H1 :bold)
  (set-color header-panel Color/BLACK)
  (set-text refresh-panel (date-format "YYYY-MM-dd HH:mm" (Date.)))
  (set-color refresh-panel Color/BLACK)
  (set-content content-panel (render-chart :bar data))
  (set-color notification-panel Color/GREEN)
  (set-font notification-panel :H3 :italic :bold)
  (set-text notification-panel "Chart is completed"))
```

**After**

``` clojure
(defn pie-chart [data layout-manager]
  (set-header layout-manager "Pie Chart")
  (refresh-date layout-manager)
  (set-content (render-chart :pie data))
  (set-notification layout-manager "Chart is completed"))

(defn bar-chart [data layout-manager]
  (set-header layout-manager "Bar Chart")
  (refresh-date layout-manager)
  (set-content (render-chart :bar data))
  (set-notification layout-manager "Chart is completed"))

(defn set-header [layout-manager text]
  (let [header-panel (:header layout-manager)]
    (set-text header-panel text)
    (set-font header-panel :H1 :bold)
    (set-color header-panel Color/BLACK)))

(defn refresh-date [layout-manager]
  (let [refresh-panel (:refresh layout-manager)]
    (set-text refresh-panel (date-format "YYYY-MM-dd HH:mm" (Date.)))
    (set-color refresh-panel Color/BLACK)))

(defn set-content [layout-manager]
  (set-content (:content layout-manager) (render-chart :bar data)))

(defn set-notification [layout-manager]
  (let [notification-panel (:notification layout-manager)]
    (set-color notification-panel Color/GREEN)
    (set-font notification-panel :H3 :italic :bold)
    (set-text notification-panel "Chart is completed")))
```

Two large functions was splitted to 6 small functions.
It was not only extracting duplicating code to functions.
Dependency was removed. Instead of passing all panels to chart rendering method, we just passing mediator `layout-manager`, which is responsible for specific rendering. Another good part is Pedro achieved what Dale asked. Chart callers depends only on layout-manager, common API. But common API implementation depends on `JCharts` which is will be replaced soon... 

### Episode 9: Observer

*Karmen:* Hi all. Good news: number of users with subscription is growing.
*Rage Man:* Excellent!
*Karmen:* Yes. We need somehow to show respect to our clients.
*Pedro:* New cat image on the front page?
*Karmen:* No, the old one is great.
*Pedro:* Cat style avatars?
*Rage Man:* Pedro, did you read requirements for future release?
*Pedro:* No, where is it?
*Rage Man:* I've sent a link to all.
*Pedro:* Didn't get it.
*Rage Man:* Interesting...
*Karmen:* I'll explain. Cat style avatars will be paid feature.
*Pedro:* Oh... I remember.
*Karmen:* Great. Any ideas about respect to clients?
*Dale:* What if we just said thanks to him?
*Karmen:* Just thanks? I'll think about it.
*Rage Man:* Yeah, not just thanks. We'll send them an email with thanks.
*Karmen:* Email is good. I will do a stub.
*Rage Man:* Super. Get to work.

*"Easy"*, thought Pedro and just called send-email function after user
got a subscription.

``` clojure
(defn subscript [user]
  (swap! user assoc :state :subscription)
  (send-email (:adress user) "Thanks!" message-body))
```

Few days he was not disturbed by anyone. Until Dale came.

*Dale:* What do we have?
*Pedro:* Everything is done.
*Dale (looking):* Did you spend three days on that?!
*Pedro:* Emm...I also refactored some old code.
*Dale:* Nice then. Commit it and I will test it on my machine.
*Pedro:* Done.
*Dale:* Have you seen Terry today?
*Pedro:* Nope.
*Dale:* Hmm, he was missing day before and don't answer the phone.
*Pedro:* He is angry on you. What did you tell her?
*Dale:* Nothing offensive.
*Pedro:* Think about it.
*Dale:* Nothing, really?
*Pedro:* Think one more time.
*Dale:* I said, nothing!
*Pedro:* Thik really hard!
*Dale:* Shut up! I said I told her nothing offensive.
Maybe he was offended by the fact that man get bigger salay then woman, but..
*Pedro:* Did you tell her that?
*Dale:* Yes, but it is a fact. Science, bitch.
*Pedro:* Then you are a moron. It is definitely a reason. Go to her home.
*Dale:* I'll handle it.

Pedro was working on something.

*Karmen:* Hi, Pedro.
*Pedro:* Hi, Karm.
*Karmen:* How things are going?
*Pedro:* Fine, the one piece left, message body.
*Karmen:* It is already done. I sent you an email.

*Thanks for subscription!*

*Now you have following advantages over standard user*

blah blah blah

*Thanks again! Even if you haven't bought it.*

*Pedro:* What does it mean, "haven't bought it?"

*Dale:* Pedro! Why did you add send email only to subscript method?
*Pedro:* Because it is the only place where subscription process is happened.
*Dale:* What if there another methods to add subscription thta we don't handle.
*Pedro:* Then we add send-email to another method.
*Dale:* It is not the case. If we forget, one user can't receive
a thanks notification.
*Pedro:* He won't die after all.
*Dale:* Won't. But if he guess (and he is definitely guess) that another
user receive a thanks message it can break our reputation.
*Pedro:* What a child.
*Dale:* Another problem then we might decide to add another functionality after subscription.
*Pedro:* We will add.
*Dale:* And we should make sure we can change the functionality at a runtime.
*Pedro:* Uh...Ok.

;; Something


*Vaine:* Hi. What a problem?
*Pedro:* Haven't you heard?
*Vaine:* No, I was helping some PHP developer to switch to Java.
*Pedro:* Important business. But, I need somehow to track changing field.
*Vaine:* Easy enough. Just make sure there is only one mutator on such field and
call needed method there.
*Pedro:* I might want to call a bunch of methods.
*Vaine:* Call.
*Pedro:* And switch what to call dynamically.
*Vaine:* I understand. Let me think... Observer. Devinitely observer.
*Niccy:* *Nope server*
*Vaine:* Hi Niccy, do you think observer is suitable?
*Niccy:* Maybe.
*Vaine:* I think it is. Because observable **maintains a list of its dependents,
called observers, and notifies them automatically of any state changes**.
*Niccy:* *Unbelievable*, you have a list, and then you call method for each element.
*Vaine:* Not that simple.
*Niccy:* Why not?
*Vaine:* You need to register observers in that list.
*Niccy:* `add`?
*Vaine:* And you need to unregister observers.
*Niccy:* `remove`?
*Vaine:* Kind of. 
*Niccy:* Is it observer?

``` clojure
(def observers (atom #{}))

(defn register [observer]
  (swap! observers conj observer))

(defn unregister [observer]
  (swap! observers disj observer))

(defn subscript [user]
  (swap! user assoc :state :subscription)
  (map apply @observers))
```

*Vaine:* Pretty similar, what is `observer` here?
*Niccy:* Function.
*Vaine:* But it is a class in Java.
*Niccy:* In most cases it is a class with one function.
*Vaine:* Well...Yes.
*Pedro:* Thanks, seems exactly what I need.
*Vaine:* Your pleasure.
*Pedro:* One question here. What if someone adds another
function with access to user and modifies state field.
*Niccy:* Cut his fingers off.
*Vaine:* No harm. You just need to agreed about contracts, 
encapsulation and style. Because it is unsolvable problem.
*Niccy:* Nothing is unsolvable...
*Vaine:* Well, proof then.

``` clojure
(def user (atom {}))

(add-watch user :state)...
```

;; TODO

**Demo**

*Sven Tori:* Sending emails says about our level.
*Rage Man:* Actually after subscription we can do a lot of things.
*Sven Tori:* For example?
*Rage Man:* We modifying subscription statistics, sending emails to me and Karm,...
*Sven Tori:* It is good. I have a proposal. We can participate in Venture Summit.
It is fame for you, reputation and you know.
*Rage Man:* Looks good to me. Anyone retort?
*Silence*
*Rage Man:* Then fine.
*Sven Tori:* Yes, the one problem you should expose your functionality for
venture qa team. They select candidates based on their testing.
*Rage Man:* Fair enough.
*Sven Tori:* When you could provide a working beta for usage?
*Rage Man:* It is already usable!
*Sven Tori:* Good then. Here's a contacts, register please.

### Episode 10: Chain of responsibility

*Rage Man:* Pedro, there are some problems occured during testing.
*Pedro:* What problems?
*Rage Man:* Something critical. I don't know the details, contact directly to their QA.
*Rage Man sent skype contacts*

;; TODO stupid qa names

*Pedro:* Hi, QA1.
*QA1:* We cannot login.
*Pedro:* Did you registered?
*QA1:* No, we  have already registered account, Rage Man gave us.
*Pedro:* Could you give me this account, I'll try on my side.
*QA1:* Not sure if it is allowed, let me check.
*Pedro:* Come on. I trying to solve problem.
*QA1:* We will ask Rage Man directly.
*Pedro:* Oh...
*Three hours gone*
*QA1:* He allowed. Here is the credentials: username/password = rageman/123
*Pedro:* Oh my god, I should add a password strength indicator... later.

Pedro succesfully logged in. All works fine.

*Pedro:* I succesfully logged in. All works fine.
*Narrator:* Sorry, Pedro. But I am a Narrator, don't repeat my words.
*Pedro:* Ok.
*QA1:* What did you do?
*Pedro:* Just entered username and password you gave.
*QA1:* Interesting, but we can't to login.
*Pedro:* I will take a look a logs.

Suddenly Pedro remebered there are no logs.
His "TODO add logging" is still a TODO.

;; Thinking
;; Dale comes, Terry Story
;; Dale logging

;; Vaine Niccy battle

``` clojure
(defn chain [[f & fs] & args]
  (when f
    (apply f args)
    (recur fs args)))
```

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








### Cast

*Should be read in style of "in the galaxy far away..."*

With the lack of imagination all characters
and names are just anagrams.

**Pedro Veel** - Developer  
**Eve Dopler** - Developer  
**Serpent Hill & R.E.E.** - Enterprise Hell  
**Sven Tori** - Investor  
**Karmen Git** - Marketing  

**Mech Dominore Fight Saga** - Heroes of Might and Magic  
**JCharts** - chart.js
