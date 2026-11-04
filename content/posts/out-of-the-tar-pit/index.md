---
title: "Out of the Tar Pit - The Good Parts"
date: 2014-03-31
description: "My notes on the paper Out of the Tar Pit: why shared state makes code hard to change, and when functional programming helps or gets in the way."
featureAlt: "Impressionist painting of wind-blown yellow flowers against a grey-blue sky."
summary: ""
tags: ["functional", "state", "assumptions", "caching"]
---

If you have been programming for a while and have not burned out yet, you know that our industry changes faster than almost any other. Every day someone comes up with a new way of programming: a new style, a new paradigm, a revolutionary new language or library.

You could say this reflects the infancy of programming. Compare it with an older engineering discipline like construction, and you will see how the centuries helped it mature. Nobody debates how to build a wall. Walls are straight and corners are square. The mathematics of Pythagoras and Aristotle, once reserved for lofty philosophical debates about celestial bodies, is now common knowledge for a high school laborer.

Programming, by contrast, is still young. It is like young wine: you do not yet know what to think of it. It is full of endless debates, and functional programming is one of the hottest.

The renaissance of functional languages began after the new millennium with Scala (2003) and Clojure (2007). The main motivation behind them is to deal with the problems of big systems and the cloud. Big systems become harder and harder to maintain as their codebases grow, and each new feature is more difficult to implement than the last. Much has been written about this in classic imperative languages, where the usual answer is a library model (dll, osgi). I was eager to find out what this fresh breeze of functional programming had to offer.

## State
[Out of the tar pit](./paper.pdf) is one of the classic functional papers on this topic that was not written in the 80's :-). I especially liked its tone: the authors did not need another alphabet or prefixed variables to express their thoughts. It is very easy to read :)

Let's start with simple request handling.
```java
public static ThreadLocal usersCache = new ThreadLocal();

public void handle(HttpRequest req, HttpResponse res) {
    String task = request.getParameter("task");   
    String userName =  req.getSession().getAttribute("userName");
    
    cacheUser(userName);
    
    try {
    	doAuthorization(req);
    	doTask(task);
        doOther();
    } finally {
    	usersCache.evict();
    }
}

private void cacheUser(String userName) {
	if (userName != null) {
    	User u = database.loadUser(userName);
        usersCache.put(u);
    }
}

public void doAuthorization() {
	User u = usersCache.get();
    //authorize user
    ...
}

public void doTask(String task) {
	//do task
    if (task == "news") doNews();
    ...
    //log action
    User u = usersCache.get();
    log(u+ " performed task " + task);
}
```

This naive pseudocode loads a user from the database, caches it in a thread-local and runs authorization followed by the task. The user info sits in a static variable, so every part of the application can use it. What is wrong with this code? In this simple example, nothing. You can safely assume that each request is handled for a single tenant identity, since a user belongs to one organization within a request.

But there is something treacherous here that is not visible at first sight. The problems begin as the code evolves. Say our product is moving to multitenancy, where a single user has a different identity and a different set of rights in each tenant (organization).

```java
private void cacheUser(String tenant, String userName) {
	
	if (userName != null && tenant != null) {
    	User u = database.loadUser(tenant, userId);
        usersCache.put(u);
    }
}
```

Our first naive implementation might look like this: load the tenant as well, create the user identity and cache it. Done! All seems well until your boss comes to your desk and asks you to show all the news from all of a user's tenants. Now you have a problem :). The news task violates the basic assumption we made earlier, that a user is in a single tenant during a request. But do not worry, you just have to bend it somehow.

```java
public void doNews() {
	User u = usersCache.get();
	String[] tenants = u.getTenants();
    for (String t : tenants) {
    	cacheUser(t, u.getUserName());
        //load news
        ...
    }
}
```

You change the cached value for each tenant and you are done! It seems fine until your boss comes back, and your code becomes more and more infested with knowledge about caching.

The main problem here is managing the state (usersCache) and its scope (a static variable, accessible everywhere). Every part of the code that touches this value has to know that it is a cached value for a single tenant identity and treat it carefully. As the system grows, so does the number of such assumptions, and your work turns into constantly solving a Rubik's cube :). You spend more and more time checking whether you have broken any of them, until the code becomes unmaintainable.

The same goes for tests. They have to cover more and more obscure situations, depending on which static state your code is in. And do not forget the cleanup afterward.

This is the main idea of the paper: do not share unnecessary (accidental) state across your code. Make all your business logic functions "pure", meaning that they do not depend on any state. All inputs are passed as arguments, and the computed value is returned rather than stored "somewhere". This approach has a huge advantage.

> Pure functions are easy to test.

## State again

As we have seen, state can become hard to maintain when it is not managed. Functional languages offer a solution by deliberately making state harder to use than a simple assignment in an imperative language. They prefer pure functions, but any real-world application still has to maintain some state. The question is how.

```clojure
;; managed references
(def bob (ref 238))
(def alice (ref 99))

;; transactional "money move"
(defn moveMoney [source dest amount]
  (dosync
    (if (>= @source amount)
      (do 
        (alter source - amount)
        (alter dest + amount))
      (throw (Error. "Out of money :-0 !")))))

;; make a money move
(moveMoney bob alice 65)

@bob
=> 173

@alice
=> 164 
```


Clojure, for example, uses software transactional memory (STM) to handle state mutation. For this it provides language primitives with different semantics: ref, agent and atom. Our example uses ref, which is a pointer to the current value.

You cannot change a ref outside a dosync block without getting an error, so state can only be mutated in the transactional manner dosync provides. It also guarantees atomicity: the money either moves from one account to the other, or the operation fails after reaching a retry threshold. Yes, it retries! That is why the mutation function has to be pure and free of side effects, so that it can safely run again. For more details, I recommend this brilliant presentation by [Rich Hickey](https://www.infoq.com/presentations/Value-Identity-State-Rich-Hickey), the author of Clojure.

## Yet, another paradigm
Any usable functional language can be divided into two parts: pure functions and some mechanism for managing state in a sane way, such as monads or STM. Seen this way, that mechanism is a restriction on state. You cannot propagate state as you please, and your code has to follow the paradigm.

Such restrictions are not limited to functional languages. Take another paradigm, OOP, which is a restriction on virtual functions. You can apply OOP in ANSI C, a strictly procedural language, but you have to do it all yourself. There is no language support for polymorphism or object inheritance, so you must construct and pass the right function pointers on your own. In the same way, functional state paradigms can be applied to an imperative language. One could argue that this would look horrible, and it surely does when done incorrectly.

Scala was designed to overcome this, and Java took a first step with lambdas, which add power and remove the unnecessary code of anonymous classes.

To be clear, Scala is neither a purely functional nor a purely OOP language. It can be tempting to think otherwise, but remember that a paradigm is a restriction. State in objects breaks the functional paradigm, and functions not bound to any object break the object-oriented one. Scala is a hybrid that adds power rather than restricting it. So be careful, because power corrupts, and this popular new language will not stop you from making an even bigger mess.

## Does Multi Core Tend to be Functional?
[One could argue](https://www.fpcomplete.com/blog/2012/04/the-downfall-of-imperative-programming) that functional languages will sooner or later replace imperative ones because they restrict state mutation. They make you write most of your code immutably and handle the remaining state with well-defined abstract structures (ref, agent, atom, monads).

So what is the problem? At first sight, again, there is none :), but ...

> Abstraction works well until it doesn't.

The problem is performance. What would you do if your superior product, written according to the latest Clojure trends, does not perform as well as you need? Say performance is a top priority, as it is in a trading messaging system, where you must buy and sell shares with the lowest possible latency.

You may find that CPU contention is now your real problem, and that threads sit blocked in a queue rather than using the full capacity of your multicore machine. You may also find that some of Clojure's language primitives are built on [JVM blocking synchronization primitives](https://www.youtube.com/watch?v=VBnLW9mKMh4). These require context switching and leave the final order of threads to the operating system. And do not forget garbage collection, since there is still a JVM underneath :).

There are several ways to solve this, and none of them depends on whether your language is functional. One is the disruptor, used by the [LMAX trading system](https://martinfowler.com/articles/lmax.html). Their approach is simple and elegant. A single thread runs the logic, so no synchronization primitives are needed at all, and another thread handles non-blocking IO. A careful reader will object that you still need a blocking operation to pass request objects from the input thread to the logic thread. Indeed, this is the tricky part, and it is where the disruptor comes in.

> The disruptor is an eventually consistent circular buffer of fixed size with single writer and possibly multiple readers.

The crucial part is the single-writer requirement. With only one writing thread, you can use the [lazySet](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/atomic/AtomicInteger.html#lazySet(long)) function of an atomic, which mutates an integer without blocking at all because [it is eventually consistent](https://psy-lob-saw.blogspot.com/2012/12/atomiclazyset-is-performance-win-for.html).

The other important detail is the fixed size, which deals with garbage collection in the simplest way: by not producing any garbage. You allocate everything at the beginning and reuse it.

At this point every functional geek should be screaming: reuse! It breaks the basic paradigm of immutable structures. True, you would not do this every day, and you chose a functional language precisely to rule it out. You should not do it in a program written in the functional paradigm without a good reason, but performance is such a reason. Apache Storm, for example, is written in Clojure, yet in version [0.8](https://storm.incubator.apache.org/2012/08/02/storm080-released.html) it replaced blocking queues with the disruptor for inter-thread communication.

> The internals of Storm have been rearchitected for extremely significant performance gains. I'm seeing throughput increases of anywhere from 5-10x of what it was before.

Sometimes you have to break a paradigm to gain extra speed. Do not let paradigms restrict you when you have a good reason, but be careful.

## Functional, the only way?
Let's rewrite all our code in a functional language! Is that the only way? Haskell will redeem us!!! Of course not. You can write crappy code in any language :).

After my enlightenment with the "functional way", I see the KISS rule in a different light. Every developer should watch for code infested with accidental state, and evict it from their project.

To be or not to be functional? It depends, and for the same reason as always. A programming language is not a matter of taste. It should be the best solution for your business case. If you choose an imperative language, you have to be careful about state, just as you have to manage memory yourself in C. With great power comes great responsibility, and people are usually bad at it. That’s all.
