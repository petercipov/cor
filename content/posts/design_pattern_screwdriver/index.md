---
title: "Design Pattern: Screwdriver"
date: 2014-11-27
description: ""
summary: ""
tags: ["soft skills", "people", "engineering", "assumptions"]
---

Programming is not entirely about coding, design and architecture. A great deal of it is also about people we are working with. I often realize that I struggle more with ego than with reason, especially when it comes to senior developers with "their ways".

One of the common problems you can encounter is the problem of a screwdriver. You have to attach the shelf with hex screws, but you do not own a proper screwdriver. What would you do? Would you use an improper one? No, you would go to the shop and obtain a proper one. It is logical.

But often you have to deal with something different. You have a bunch of programmers that do not see why you need this different solution when the "improper" screwdriver worked just fine yesterday. You are trying to explain why, but it is not accepted, for many odd reasons, like "I don't believe it will work better". Of course there are many reasonable [problems with new technology](http://techblog.netflix.com/2014/11/nodejs-in-flames.html), you have to mitigate risk beforehand. But sometimes the problem is not in technology or business but in people themselves, more precisely in their egos.
You silently say jeeesh, and fight your struggle because you know where it would end otherwise. You would cripple a screw. And any future change will be nearly impossible.

If you like your job, you would definitely avoid this even though you would become the black sheep. Your main motivation is the product itself. You know that any change to a dilettante design would be costly and drastic, therefore it is impossible to get rid of it completely. Making change hard is the guaranteed path of doom.

> The only constant thing in programming is change.

I have encountered this "back-pressure" numerous times with TDD, but the absolute peak of resistance was reached when I refused to use an "in-house framework" that was someone else's toy, developed for years. I had become that bad colleague who has taken someone's toy. Something mind-blowing for me at that time.

And I have seen this nearly everywhere where something (framework, library, tooling) was developed for some time. The lack of will to change favourite tools to proper ones, fitting the current situation better.

> For several years I was the lone C programmer in that company. I thought that everybody should be programming in C. But I could not convince any of my brethren to start programming in C. Does anybody have that problem? With something like test driven development or agile practices? The most common questions that people would ask me when I do lecture on TDD, is how can I convince anybody else to do this. And the answer to this question is you can not. [Robert Martin](http://www.infoq.com/presentations/clojure-c)

Somehow I could not understand why such smart people are able to make such bad decisions. The journey for this answer has taken me far from my domain of expertise, which is programming, to psychology, namely to the [Dunning-Kruger effect](http://en.m.wikipedia.org/wiki/Dunning%E2%80%93Kruger_effect).

Their initial test study was the case of bank robber McArthur Wheeler:

> In 1995, McArthur Wheeler walked into two Pittsburgh banks and robbed them in broad daylight, with no visible attempt at disguise. He was arrested later that night, less than an hour after videotapes of him taken from surveillance cameras were broadcast on the 11 o'clock news. When police later showed him the tapes, Mr. Wheeler stared in incredulity. "But I wore the juice", he mumbled. Apparently, Mr. Wheeler was under the impression that rubbing one's face with lemon juice rendered it invisible to videotape cameras.

The experiment was done on Cornell University undergraduate students. They were tested on various topics like humor, logical reasoning and English grammar. Then they had to write an estimate of their abilities: how successful they were compared to the others. They did not have a clue about the results of the others. It was only their guess.

In their famous paper [Unskilled and unaware of it](./paper.pdf)  they argued that:

> People are incompetent in the strategies they adopt to achieve success and satisfaction, they suffer a dual burden: Not only do they reach erroneous conclusions and make unfortunate choices, but their incompetence robs them of the ability to realize it.

<figure>
  <img src="duning.png" >
  <figcaption>
    Top performers underestimate themselves, under performers overestimate their competence.
  </figcaption>
</figure>

* Students with the lowest score (bottom quartile) overestimated their results
* Students with the highest score (top quartile) underestimated their results
* Estimates of top quartile students have improved when they were confronted with the work of the others. (They checked tests of other participants.)
* Students from bottom quartile have problems realizing their incompetence even after the confrontation with the results of the others.

> The competent do not know they are competent and the incompetent do not realize their incompetence.

And sometimes this is the exact situation you are facing. Sometimes you work with people with superior skills but they do not use them fully because they underestimate themselves. I have heard many times we are not big enough, we cannot compete with giants like Google or Facebook. You somehow understand them but then you realize that big teams do not work and any superior feature you are seeing these days was made by small teams, same as yours. If you have skillful people why not use their full potential, take the risky way and go into uncharted waters? Nevertheless, this gives you the ability to stretch your capability to a point beyond competition.

> People tend to do what they are used to, programmers are no exception to this.

Many times you encounter colleagues with zero motivation. They are smart, full of potential but they do not believe in the project. This aspect is underestimated. We programmers like to focus on the technological aspects of the project and underestimate everything else. They may seem inferior to the logical problems we love to solve. I used to think that algorithms are the hardest part of the software development process, but actually they are not when you are working in a team. Small things like motivation are absolutely essential when it comes to innovation. Motivation gives you the simple answer why to actually bother and learn new things and not use something that has worked "just fine" yesterday.

> It is lethal when you use tools of yesterday today to invent something that will be successful tomorrow.

You have to focus on the idea itself and not distract yourself by figuring out how to bend inappropriate tools, it may be your favourite library, framework, programming language, operating system, database system, cloud service. I have seen a transition of file storage from OpenStack Swift to Mongo GridFS, because we found out that Swift reads from disks all the time to checksum stored files and guarantee file consistency. This has led to disk failures more often than we would like to anticipate. Hardware is really costly and I did not mention delays due to logistics - someone has to buy disks and install them. Grid seems fine from the beginning, no nonstop disk checksums, until it starts to lose data due to software bugs. Try to explain this to your customer :). Then you realize that cloud storage providers like Amazon offer you storage as a service with no additional costs than monthly fee. You realize that storing data in big quantities is not as easy task as it seemed at the beginning. Storing data is not your core business (this business has already been dominated). From this point of view migration to Amazon is very attractive. But it depends on your business, you will overpay if you are storing a few GB and storing half of the internet is more expensive than the custom solution. Therefore you should know your tools perfectly and also explore other possibilities. Maybe your problems are already solved and you are reinventing the wheel.