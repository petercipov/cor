---
title: "Religious issue: The best programming language"
date: 2013-11-17
description: ""
summary: ""
tags: ["engineering", "languages", "assumptions"]
---
What is the best programming language? I have spent many hours in this endless debate and the conclusion was that for many programmers it is a matter of taste.

Today the sexiest language is JavaScript & Node.js platform. Tons of new projects are popping up like a fresh breeze. The new zeitgeist is "all in JavaScript". It is easy, it is fast and event-loop will redeem your project. You write your client/server code in a single language. So from now you just need JavaScript coders.

<figure>
  <img src="github_languages.png" >
  <figcaption>
    Github repositories language chart.
  </figcaption>
</figure>

Really? Is it really a matter of taste? Of course not. A programming language should always be a matter of your current circumstances, the workload you are solving.

If you want to scale, it is a bad idea to write database drivers purely in JavaScript. Think first about garbage objects that this will produce. At low scale it is ok but soon you will reach a technology limit. Then you can choose whether to buy a new machine or optimize your code. It is not an obvious choice. You can use a well-written low-level driver in C. You will wrap it with JavaScript and create an npm package. Everything seems fine again until bugs pop up and you cannot debug into the linked C library from JavaScript. Your programmer is speechless and wants you to find someone to fix it. You can hire some C freak or you can just buy more and more machines. Sometimes a new machine is quite enough, sometimes TCO (total cost of ownership) is bigger.

Write your code once - client and server in a single source base. It is a myth. I have written in JavaScript for a few years and there were numerous cases when IE sucked, when Firefox had some bug, when Chrome went crazy. On the other hand you want to support the widest range of browsers, so you have to ify (add if) your code. You have multiple bent places here and there to overcome that or other browser weak part - it is better not to share this. Maybe in the future this will be fixed but now this sucks.

Wheel of reinvention - Using JavaScript everywhere is a trend. It may seem that they solve a new set of problems. They do not. For example sockets have been known since the beginning of network communication, but in JavaScript it is a new HTML 5 addition with the new name WebSockets. In fact JavaScript greatly solves just a single problem and this is application distribution. No complicated installation, but you can bring a similar user experience as native apps with one click. Therefore browsers are becoming minimalistic, Chrome is just a single toolbar, so there is more place for your application.

JavaScript is pure evil - This is also a common opinion. JavaScript is slow. You have to download MBs of JavaScript to render the DOM. You should instead render HTML on the server. On the other hand there are multiple reasons why you do not want to do this. When you divide presentation layer from server business logic you can develop both sides separately. You specify an API between them, e.g. via apiary.io, and then you can develop in parallel. Second reason is even more important. You can make your applications rich. It is not a set of isolated pages anymore. With HTML 5 JavaScript you can create a hell of a user experience. This is a paradigm shift. Internet connection is fast so you can transfer MBs of JavaScript to your single page app. You often do this only once and then the internal cache of the browser will return the stored code without a download from the server. JavaScript is a very powerful tool, but if you give a knife to a small child, they will cut themselves.

Always think about your problem first. What are your cases? Will language X solve some of them? It should not be a matter of personal preferences or feelings. I think ..., I hope ... are often signs of assumptions and bias. You should avoid them in your project. You can easily find someone with different opinion in your team. Endless debates will make you tired without any gain of business value. You lose time and money. Software is a risky business. Can you afford to burn your opportunities instead of delivering value?

> There is no best programming language. There is an optimal language for your job.
