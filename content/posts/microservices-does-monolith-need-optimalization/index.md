---
title: "Microservices - Does a monolith need optimization?"
date: 2015-04-20
description: "Microservices aren't always the answer. When a simple monolith is fine, when it starts to hurt, and how to split it into smaller services."
summary: ""
tags: ["microservices", "assumptions"]
---

Micro-services is the buzzword of today. It is the new best, something you should definitely implement. No! Never believe such a fancy statement. Engineering is never sunshine and roses. You should be able to reason about problems and choose what is best for you even though it is not popular at the moment.

What is a monolith? According to the [dictionary](http://www.oxforddictionaries.com/definition/english/monolith) it is a statue built from a single piece of stone. In our case we can imagine a classic type of application written in a single language, stored in a single repository, executed in a single process, using a single database, with a single view model.
Most monoliths have in common:

* coupled features - one feature cannot exist without another
* spaghetti code - there are no clear boundaries, code is fuzzy, illogical, full of weird names like Provider, Factory, Manager, Abstract, Executor.
* global mutable shared state is a superstar.
* no decomposition

These are not really disadvantages of a monolith, but these situations are just very common, because it is so easy to do them. And when deadlines are near it is almost a rule to bend something, to adjust something, just to make it work!

<figure>
  <img src="monolith.png" alt="Graph of nodes where nearly every node is connected to every other node." >
  <figcaption>
    Example of the monolith.
  </figcaption>
</figure>

We can imagine a monolith as a graph where nodes are functions, classes, libraries, packages, data storages, tables and edges are the dependencies between them. This image is an example of a coupled monolith, it is a nearly full graph, where everything depends on everything. This is a typical nightmare when you obtain code after your colleague, you are not even able to understand such a beast :) and this one was maintained for decades. I have seen projects where it took months for developers just to read over the codebase and understand what it does. I do not need to tell you how much joy such work brings.

What will happen when your project starts to grow? The number of nodes and edges will also grow. Such a project requires more manpower and so you hire more people and now there are multiple of you that can change something. Just to change the public interface of a single node you will have to repair all of its dependencies and the situation becomes even worse when a change will "bubble" to multiple levels of the graph. Changing something is a nightmare and, even worse, in time the graph will grow and so does the complexity.

Doing optimisation is sometimes impossible. Let's imagine you have a single table where you store structure and data. Structure is for example tree structure where every table row has a pointer to its parent and one huge blob of data. Writing optimal SQL for such a table is nearly impossible. It really does not matter what kind of database storage you use. You want the structure to be cached in RAM, but big blobs flush it and make it less effective. What you really need is to divide structure and data and optimise storage for each of them.

To be honest, on the other hand a monolith is so easy to start with that it is almost no-brainer for small projects in their early stages. In this state you do not even know what your project will look like. Leave micro-services for later and focus on business value instead.

> If you don’t end up regretting your early technology decisions, you probably over engineered……Re-architecting is a sign of success; if you never need to, either you overbuilt or nobody cares. (Randy Shoup)

Lots of projects start as monoliths and a monolith is just fine for them for a very long time. But then comes a time when new features require something that a monolith is not able to provide without hacks. For many cloud services it is high availability. In simple words you cannot just shutdown your service, upgrade all binaries, run an upgrade of the database overnight and pray that in the morning everything will work. You have to chop the monolith into a set of services and upgrade them separately, so when you shut down something you shut down partially or in better case you just degrade responsiveness. Upgrade can be managed and that is the core idea. You are in control which part is upgraded and you choose proper steps and order in which they are applied.

```
eBay
Monolith in Perl   -> Monolith in C++ -> Java    -> microservices

Twitter
Monolith in Rails  -> JS / Rails / Scala       -> microservices

Amazon
Monolith in C++    -> Perl/C++   -> Java/Scala -> microservices
```


All internet giants are now running clusters of micro-services. They have started as monoliths. They were written mostly in the favourite language of the company founder. In time, monoliths were rewritten into better structured monoliths and finally chopped into ecosystems of micro-services. What became important was the possibility for services to communicate easily. This was the reason for the creation of Facebook Thrift, Google Protobuf and REST. Services written in Python can easily communicate with services written in Java or C.

<figure>
  <img src="monolith-chop.png" alt="The same tightly connected graph, with an arrow pointing to a version that has far fewer connections between nodes." >
  <figcaption>
    Example of the monolith chopping.
  </figcaption>
</figure>


The solution to problems with a monolith is quite trivial. Chop it into services and manage them with a version control system. Just like we are used to with maven, osgi, npm, the same approach was reused for micro-services (here comes handy docker and its image versioning).
Every micro-service is responsible for a single function, for its data and upgrades. A micro-service does just one thing (just registration, just login, just authorisation or just comments). What you will see in time is gradual chopping. You start with a single service and gradually you chop. You can chop literally till you drop. :)

Top-down design is the rule of thumb when you do so. You start from your requirements and divide them into smaller parts until you provide optimal solution for them. The technology stack should be the last thing you choose. If you need NoSQL storage, fine, use it. If MySQL is enough, fine, use it. But do not do it in reverse order: I have MySQL and PHP, Drupal, and how would I write that feature in it? You are adding restrictions on yourself and making your task harder than it needs to be.

I have seen an overkill solution for the famous Java EE perm-gen exception - Java EE containers will eventually crash when you deploy WAR files multiple times, because loaded jars and classes are not garbage collected. The proposed solution will look at your WAR file, process classes and suggest how to bend your code to actually prevent it. It seems useful. But do you really need WAR deployment? Do you need a full container? If not, what about embedding container in your code? Run a simple Main class that starts containers in embedded mode. From this moment you have no problems with perm gens!

> The simplest solution to any problem is ... do not create the problem in the first place.

If it is problematic for you to properly structure a monolith, micro-services will be hell.

Looking for the reason why something fails on a distributed production system is hell. You do not have a single process that can be broken, but you have dozens or hundreds of them.

Therefore writing micro-services without tests on every level is a no-go. Cover your code with unit tests and gradually write integration tests. You will soon realise why coding without them is a bad idea. It will take you days to actually find out why something does not work!

The approach of micro-services is an optimisation. The rule of thumb for optimisation - do not optimise if you do not need to - also applies for micro-services. Do not use them blindly. They can help a lot but they can be a nightmare if they are done badly.

P.S. I am waiting for the stupid discussion that will arise in time: micro-services are dead, long live the monolith. I have seen the same fallacy with "TDD is dead". The disappointment of developers who actually did not understand the topic well enough to find out that nothing is a silver bullet and everything has its good and weak parts.