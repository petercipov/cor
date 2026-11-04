---
title: "Microservices - Does a monolith need optimization?"
date: 2015-04-20
description: "Microservices aren't always the answer. When a simple monolith is fine, when it starts to hurt, and how to split it into smaller services."
featureAlt: "Painting of a bridge over a pond of water lilies."
summary: ""
tags: ["microservices", "assumptions"]
---

Micro-services are today's buzzword, the new best thing that you should definitely implement. No! Never believe a statement like that. Engineering is never all sunshine and roses. You have to reason about your problems and choose what is best for you, even when it is not fashionable.

What is a monolith? According to the [dictionary](http://www.oxforddictionaries.com/definition/english/monolith), it is a statue carved from a single piece of stone. In our case, it is a classic application with a single language, repository, process, database and view model.

Most monoliths have these things in common:

* coupled features: one feature cannot exist without another
* spaghetti code: no clear boundaries, and fuzzy, illogical code full of weird names like Provider, Factory, Manager, Abstract and Executor
* global mutable shared state as the superstar
* no decomposition

These are not inherent disadvantages of monoliths, but they are very common because they are so easy to fall into. When deadlines approach, it is almost a rule to bend or adjust something just to make it work!

<figure>
  <img src="monolith.png" alt="Graph of nodes where nearly every node is connected to every other node." >
  <figcaption>
    Example of the monolith.
  </figcaption>
</figure>

Imagine a monolith as a graph. The nodes are functions, classes, libraries, packages, data stores and tables, and the edges are the dependencies between them. The image above shows a tightly coupled monolith: a nearly complete graph where everything depends on everything. This is the typical nightmare you inherit from a colleague, a beast you cannot even begin to understand :), and this one was maintained for decades. I have seen projects where it took developers months just to read through the codebase and understand what it does. I do not need to tell you how much joy such work brings.

What happens when your project starts to grow? The number of nodes and edges grows too. The project needs more people, so you hire them, and now more of you can change things. Changing the public interface of a single node means repairing all of its dependencies, and it gets even worse when the change "bubbles" through several levels of the graph. Every change is a nightmare, and as the graph grows, so does the complexity.

Sometimes optimization becomes impossible. Imagine a single table that stores both structure and data, for example a tree in which every row has a pointer to its parent and one huge blob of data. Writing optimal SQL for such a table is nearly impossible, whatever kind of database you use. You want the structure cached in RAM, but the big blobs keep flushing it out. What you really need is to separate the structure from the data and optimize the storage of each.

On the other hand, a monolith is very easy to start with, which makes it almost a no-brainer for small projects in their early stages. At that point you do not even know what your project will look like. Leave micro-services for later and focus on business value instead.

> If you don’t end up regretting your early technology decisions, you probably over engineered……Re-architecting is a sign of success; if you never need to, either you overbuilt or nobody cares. (Randy Shoup)

Many projects start as monoliths, and a monolith serves them well for a very long time. Eventually, though, new features require something a monolith cannot provide without hacks. For many cloud services, that feature is high availability.

You cannot simply shut down your service, upgrade all the binaries, upgrade the database overnight and pray that everything works in the morning. You need to split the monolith into services and upgrade them separately. When you shut something down, only part of the system goes down, and at best you only degrade responsiveness.

The core idea is that upgrades become manageable. You control which part is upgraded, and you choose the steps and the order in which to apply them.

```
eBay
Monolith in Perl   -> Monolith in C++ -> Java    -> microservices

Twitter
Monolith in Rails  -> JS / Rails / Scala       -> microservices

Amazon
Monolith in C++    -> Perl/C++   -> Java/Scala -> microservices
```


All the internet giants now run clusters of micro-services, but they started as monoliths, mostly written in the founder's favourite language. Over time, those monoliths were rewritten into better-structured ones and finally split into ecosystems of micro-services. Easy communication between services became essential, which led to Facebook Thrift, Google Protobuf and REST. A service written in Python can now easily talk to services written in Java or C.

<figure>
  <img src="monolith-chop.png" alt="The same tightly connected graph, with an arrow pointing to a version that has far fewer connections between nodes." >
  <figcaption>
    Example of the monolith chopping.
  </figcaption>
</figure>


The solution to the problems of a monolith is quite straightforward: split it into services and manage their versions. We already do this with maven, osgi and npm, and the same approach works for micro-services, with docker and its image versioning coming in handy.

Each micro-service is responsible for a single function, along with its data and upgrades. It does just one thing: registration, login, authorization or comments. Over time, the splitting becomes gradual. You start with one service and keep chopping. You can literally chop till you drop :)

As a rule of thumb, use top-down design. Start from your requirements and divide them into smaller parts until you can solve each one optimally. The technology stack should be the last thing you choose. If you need NoSQL storage, fine, use it. If MySQL is enough, use that. Just do not work in reverse, starting from "I have MySQL, PHP and Drupal, so how do I write this feature?" That only restricts you and makes the task harder than it needs to be.

I once saw an overkill solution to the famous Java EE perm-gen exception. Java EE containers eventually crash when you deploy WAR files repeatedly, because the loaded jars and classes are not garbage collected. The proposed tool would inspect your WAR file, process its classes and suggest how to bend your code to prevent this. That sounds useful, but do you really need WAR deployment or a full container? If not, why not embed the container in your code? Run a simple Main class that starts it in embedded mode, and your perm-gen problems are gone!

> The simplest solution to any problem is ... do not create the problem in the first place.

If you struggle to structure a monolith properly, micro-services will be hell.

Finding out why something fails on a distributed production system is hell too. Instead of one process that can break, you have dozens or hundreds.

That is why writing micro-services without tests at every level is a no-go. Cover your code with unit tests and gradually add integration tests. You will soon see why coding without them is a bad idea, when it takes you days to find out why something does not work!

Micro-services are an optimization, and the rule of thumb for optimization applies to them as well: do not optimize unless you need to. Do not adopt them blindly. They can help a lot, but done badly they can be a nightmare.

P.S. I am waiting for the stupid discussion that will come with time: micro-services are dead, long live the monolith. I have seen the same fallacy with "TDD is dead". It comes from the disappointment of developers who never understood the topic well enough. Nothing is a silver bullet, and everything has its strengths and weaknesses.
