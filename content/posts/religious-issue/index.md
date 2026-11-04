---
title: "Religious issue: The best programming language"
date: 2013-11-17
description: "There is no best programming language. Why the right choice depends on the job you need to do, with JavaScript and Node.js as the example."
featureAlt: "Old drawing of a woman holding a child, with other figures beside her."
summary: ""
tags: ["engineering", "languages", "assumptions"]
---
What is the best programming language? I have spent many hours in this endless debate, and the conclusion is usually that, for many programmers, it is a matter of taste.

Today the sexiest choice is JavaScript and the Node.js platform. New projects are popping up everywhere like a fresh breeze, and the new zeitgeist is "all in JavaScript". It is easy, it is fast and the event loop will redeem your project. You write client and server code in a single language, so from now on you only need JavaScript coders.

<figure>
  <img src="github_languages.png" alt="Line chart of new GitHub repositories by language from 2008 to 2013, with JavaScript rising to the top ahead of Ruby and Java." >
  <figcaption>
    Github repositories language chart.
  </figcaption>
</figure>

Really? Is it just a matter of taste? Of course not. Your choice of programming language should depend on your current circumstances and the workload you are solving.

If you want to scale, writing database drivers purely in JavaScript is a bad idea. Think about all the garbage objects this will produce. It is fine at a low scale, but you will soon hit a technology limit. Then you must choose between buying a new machine and optimizing your code, and the choice is not obvious.

You could use a well-written, low-level driver in C, wrap it with JavaScript and publish an npm package. Everything seems fine again until bugs appear and you cannot debug into the linked C library from JavaScript. Your programmer is speechless and asks you to find someone who can fix it. You can hire some C freak, or you can keep buying more machines. Sometimes a new machine is enough, and sometimes the TCO (total cost of ownership) is higher.

Write your code once, with client and server in a single codebase? That is a myth. After writing JavaScript for a few years, I have seen plenty of cases where IE sucked, Firefox had a bug or Chrome went crazy. Yet you want to support the widest range of browsers, so you have to litter your code with ifs. You end up with bent code here and there to work around each browser's weak spots, and you would rather not share that with the server. Maybe this will be fixed in the future, but for now it sucks.

Then there is the wheel of reinvention. Using JavaScript everywhere is a trend, and it may seem to solve a new set of problems, but it does not. Sockets, for example, have existed since the beginning of network communication, yet in JavaScript they are a new HTML 5 addition called WebSockets. In fact, JavaScript solves just one problem really well: application distribution. With no complicated installation, you can offer a user experience close to a native app with one click. That is why browsers are becoming minimalistic, with Chrome being just a single toolbar, leaving more room for your application.

Another common opinion is that JavaScript is pure evil. It is slow, you have to download megabytes of it to render the DOM, and you should render HTML on the server instead.

But there are good reasons not to do that. Separating the presentation layer from the server's business logic lets you develop both sides independently. You specify an API between them, for example with apiary.io, and then develop them in parallel.

The second reason is even more important: you can make your applications rich. They are no longer sets of isolated pages. With HTML 5 and JavaScript you can create a hell of a user experience, and this is a paradigm shift. Internet connections are fast enough to deliver megabytes of JavaScript to a single-page app. You often download them only once, since the browser cache serves the stored code after that. JavaScript is a very powerful tool, but if you give a knife to a small child, they will cut themselves.

Always think about your problem first. What are your use cases? Will language X solve any of them? The decision should not come down to personal preferences or feelings. "I think ..." and "I hope ..." are often signs of assumptions and bias, and you should keep them out of your project. It is easy to find someone on your team with a different opinion, and endless debates will only wear you out without adding any business value. You lose time and money. Software is a risky business. Can you afford to burn your opportunities instead of delivering value?

> There is no best programming language, only the best language for your job.
