---
title: "Out of the Tar Pit - The Good Parts"
date: 2014-03-31
description: ""
summary: ""
tags: ["functional", "state", "assumptions", "caching"]
---

If you are programming for some time and you still not suffer from burnout syndrome, you definitely recognize that our industry is one of the most changing and adapting industry. On daily basis someone come with the new way of programing - new style, new paradigm, new revolutionary language or library. One would say that this has something in common with infancy of programming. If we compare programming with some other engineering industry like construction, we will see that ages helped it mature. There is no debate how to build the walls. They are straight. Corners are at the straight angle. Math from Pythagoras and Aristotelian is now common knowledge of high school laborer. Even though this knowledge was originally meant only for the highest philosophical debates about celestial bodies. The fact is that programming is is still young. Its like young wine - you do not know what to think about it. Its full of these endless debates. One of the hottest is functional programming.

The renaissance of functional languages started somewhere after new millennium with Scala (2003), Clojure (2007). The main motivation behind them is to handle problems of big system and clouds. It turns out that it is harder and harder to maintain big systems as their code base grows. New features pop often and every time it is harder to implement it. There was written a lot about this topic in classic imperative languages - it is solved with libraries model (dll, osgi). I was eager to discover what this new fresh breeze of functional programming has to offer.

## State
[Out of the tar pit](./paper.pdf) is one of the classic functional papers on this topic not written in 80's :-). I liked tone of the the paper even more - authors did not need another alphabet and prefixed variables to express their thoughts. It is very simple to read :)

Lets start with simple request handling
```java
public static ThrealLocal usersCache = new ThreadLocal();

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

This naive pseudo code loads user from database, caches it in thread-local and runs the authorization followed with task execution. User info is cached in a static variable so it can be used from every part of the application. What is the problem with this code ? In this simple example nothing. You can easily postulate that request is always handled for single tenant identity (user always belongs to single organization in single request). But there is something treacherous. It is not visible on the first sight. Problems begins with code evolution. Our product and code is switching to multitenancy. Single user has different identity and different set of rights in every tenant (organization).

```java
private void cacheUser(String tenant, String userName) {
	
	if (userName != null &&| tenant != null) {
    	User u = database.loadUser(tenant, userId);
        usersCache.put(u);
    }
}
```

Our first naive implementation would like this. Load also tenant and create user identity and cache it. Done! Seems still ok until your boss comes to your desk and gives you a task providing all news from all user tenants. And now you have a problem :). News task is the violation of basic postulate we mentioned before. (User is in single tennant in request). Ok do not worry you have to bent it somehow.

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

You change cached value for every tenant and you are done! It seems fine until boss came again. And your code is more and more infested with knowledge about caching. The main problem here is management of state (usersCache) and propagation scope (static variable) of this state - it is accesible everywhere. Every part of code that is working with this value has to know that this is cached value for single tenant identity and has to work with it carefully. With bigger and bigger systems you would have more and more of these postulates across your code and your work will turn to constant solving of Rubik cube :) You will spent more and more time on whether you did not break any of those postulates until it will be unmaintainable. The same thing holds also for our tests. Your tests will have to test more and more obscure situations when your code is in that good or that bad static state and do not forget cleanup afterward.

This is the main idea of presented paper. Do not share unnecessary state (accidental state) across your code and make all your business logic functions "pure" - this means those functions do not depend on any state. All input are method variables and computed value is not stored "somewhere" but returned. This approach has a huge advantage.

> Pure functions are easily tested.

## State again

As we mentioned before maintaining state can become hard when it is not managed. Functional languages provides a way how to solve it. To be more precise they make it much more harder to for programmer than just assign statement in imperative languages. Functional languages prefer pure functions, but to construct some real world application you need to maintain state in your application. The question is how.

```clojure
;; managed references
(def bob (ref 238))
(def alice (ref 99))

;; transactional "money move"
(defn moveMonevy [source dest amount]
  (dosync
    (if (>= @source amount)
      (do 
        (alter source - amount)
        (alter dest + amount))
      (throw (Error. "Out of money :-0 !")))))

;; make a money move
(moveMonevy bob alice 65)

@bob
=> 173

@alice
=> 164 
```


Clojure, f.e. uses software transactional memory (STM) to handle state mutation. For this purpose it provides language primitives like ref, agent and atom. They have different semantics. In case of our example ref is demonstrated. It is a pointer to current value. You cannot change ref outside of doSync block otherwise you will get an error. You can only mutate state in transaction like manner that is provided with doSync. It guarantees you also the atomicity - money will be transferred from one account to another or it will fail after it reaches some threshold of retry. Yes, it retries ! Therefore mutation function has to be pure, without side effects, so it can be executed again. For more details I recommend brilliant presentation from [Rich Hickey](http://www.infoq.com/presentations/Value-Identity-State-Rich-Hickey) (author of Clojure)

## Yet, another paradigm
If you look to any usable functional language you can divide it to two parts. Something that is pure functional (pure functions) and some kind of mechanism how to manage state. This are f.e. monads or STM. It is a mechanism how to handle state in a sane way. And from this point of view you can characterize it as a restriction over state. You do not propagate state as you want but you write your code to hold the paradigm. Such restriction is not only bound to functional languages. To demonstrate this switch to another paradigm, OOP. OOP is restriction over virtual functions. In case of ANSI C that is strictly procedural, OOP paradigm can be applied, but you have to handle it by yourself. You have to construct and pass proper function pointer. There is no language support for polymorphism and objects inheritance. You have to construct and use it by yourself. In same manner functional state paradigms can implied to imperative language. One could argue that this would look horrible. And it surely is horrible if it is done incorrectly. To overcome this Scala language was designed and Java made a first step to gain extra power and to remove unnecessary code of anonymous classes with introduction of lambdas.
To be clear Scala is not functional language and it is not also OOP language. It can be tempting to think the opposite, but as it was mentioned before. Paradigm is the restriction. If you have state in objects you are breaking functional paradigm and if you have set of functions that are not bound to any object you break object oriented paradigm. It is a hybrid solution. So you are not restricting power but adding even more. So be careful. Power corrupts. And this new and popular language does not save you from making even bigger mess.

## Does Multi Core Tend to be Functional?
[One could argue](https://www.fpcomplete.com/blog/2012/04/the-downfall-of-imperative-programming) that functional programming languages will soon or later replace imperative programming languages, because it brings restriction over state mutation. It forces you to write most of your code immutably and the only only necessary state is written with help of well defined abstract structures (ref, agent, atom, monads).
Well, what is problem with that ? On the first sight nothing again :) , but ...

> Abstraction works well until it doesn't.

What I mean is performance. What would you do if you discover that your superior product that was written according to the newest trends in Clojure is not performing as you wish? Performance is one of your top priorities. If you are writing messaging system for trading, it is crucial to buy and sell shares with the lowest latency possible. You will find out that cpu contention is your real problem now. You will find out that threads are blocked an they are waiting in the queue instead of running at the maximum capacity of your multicore machine. You will find out that some of those language primitives of Clojure are written with [JVM blocking synchronization primitives](https://www.youtube.com/watch?v=VBnLW9mKMh4) that requires context switching and require operating system to decide the final order of the threads. Do not forget garbage collection, there is still JVM underneath :).

There are several ways ho to solve it. And they do not depend on whether your language is functional or not. On of them is disruptor - approach of [LMAX trading system](http://martinfowler.com/articles/lmax.html). Their approach is simple and elegant. They use just single thread for logic so they do not use any synchronization primitives at all. They use one other thread for non-blocking io handling. The Careful reader would now argue that you still need blocking operation for passing request object from input thread to logic thread. Indeed this is the tricky part. Here comes disruptor for remedy.

> Disruptor is eventual consistent circular buffer of fixed size with single writer and possibly multiple readers.

Crucial part here is single writer requirement. In case of single writing thread you can use [lazySet](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/atomic/AtomicInteger.html#lazySet(long)) function of atomic that mutate integer and do not block at all - [it is eventually consistent](http://psy-lob-saw.blogspot.cz/2012/12/atomiclazyset-is-performance-win-for.html).
The other important detail is fixed size. It is the solution for garbage collection. Do not produce garbage at all. You should allocate everything at the beginning and then reuse.

Now every functional geek should scream - reuse. It breaks the basic paradigm - immutable structures. It is true that you do not do this on daily basis. You have chosen functional language exactly to forbid this. You should not do this in program written with functional paradigm until you have a good reason and performance is such reason. For example Apache Storm is project written in clojure. But with version [0.8](https://storm.incubator.apache.org/2012/08/02/storm080-released.html) they have switched their intert-hreads communications from blocking queues to disruptor.

> The internals of Storm have been rearchitected for extremely significant performance gains. I'm seeing throughput increases of anywhere from 5-10x of what it was before.

In some cases you have to break paradigm to gain extra speed. Do not be restricted with paradigms if you have a good reason. But be careful.

## Functional, the only way ?
Lets rewrite all our code in functional language! It is the only way ? Haskel will redeem us !!! Of course not. You can write crappy code in any language :).

After my enlightenment with "functional way", I see the KISS rule in different light. Infestation of code with accidental state is something that every developer should think about and evict from his project.

So be or not to be functional. It depends. The reason is still the same. Programming language is not matter of taste. It should be the best solution for your business cases. If you choose imperative language you will have to be careful about state. It is same responsibility as with self-managing of memory in C. You are given big power and also big responsibility. People are usually bad at this. That’s all.