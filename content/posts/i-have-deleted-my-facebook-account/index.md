---
title: "I have deleted my Facebook account, what now"
date: 2018-06-08
description: "Why I left Facebook over privacy, and how I set up my own chat server with Matrix to stay in touch with family and friends."
featureAlt: "Painted portrait of a red-bearded man in a blue jacket with a crowd of people behind him."
summary: ""
tags: ["messaging", "element.io"]
---

> TL;DR: I have deleted my Facebook account. Ciao.

There is not much more to add. I asked for my account to be deleted, and I had my reasons. Facebook provides an amazing service, but it comes at a price, and that price is your privacy.

From your first log-in, every photo is analyzed, every message is read by AI and even your movements outside Facebook are tracked. Together, they form the new currency of the 21st century: your behavior, opinions, plans and desires. That is what we used to call privacy.

Today, privacy is ignored and treated as an obsolete relic that is hard to exploit. Because it is invisible, it is easy to overlook. Yet it matters enormously, because we act on our feelings over 90% of the time.

Have you ever wondered why ads on web pages can be so precise? It is because you are being tracked. Did you know that Amazon may show different prices to you and your friends? Sales teams use psychological profiles to set margins and prices for each customer. This extends even to presidential elections: Trump used psychological profiles of US citizens in his last campaign, and the pro-Brexit side used them to win the referendum in the UK. Read about Cambridge Analytica if you are interested.

The new data privacy regulation, GDPR, is now being enforced, and I welcome it because this field needs it. I see it as a regulation that protects your data, something nobody cared about until now. IT is not as mature as other branches of business. With so few regulations, it could exploit its full potential, but in my opinion it took too much and made privacy obsolete.

I do not expect my letters to be opened when I send them through PPL or DHL. These companies do not open and scan packages to sell the information to third-party partners. Would you mind if they did? If so, why do you not mind when Facebook does it? I will leave that question for readers to answer themselves.

> My mind is clear in this matter. I have decided to leave Facebook due to its "no privacy" policy. Privacy matters!

I do not want to cut off my contacts, though. So I decided to host my own chat service and gradually moved my family and all important contacts over to it. Here I want to share the details of why, what and how I did it.

## Why

My "Facexit" idea started a few months ago. My last-nail-in-the-coffin moment came when I read about Cambridge Analytica.

> Propaganda works best, when you do not know its source. Facebook delivered what propagandists have always wanted, a complete blurring of the line between editorial and advertising, often delivered with the added reliability of having been “shared” by a “friend”. [1](https://www.theguardian.com/technology/2018/mar/24/facebook-week-of-shame-data-breach-observer-revelations-zuckerberg-silence)

> ... figuring out whether a particular voter was, say, a neurotic introvert, a religious extrovert, a fair-minded liberal or a fan of the occult. Those were among the psychological traits the firm claimed would provide a uniquely powerful means of designing political messages. [2](https://www.nytimes.com/2018/03/17/us/politics/cambridge-analytica-trump-campaign.html)

What I consider fundamentally broken is the monopoly:

* Your identity is a Facebook identity. You can have only one account and communicate only with other Facebook accounts. If you dislike Facebook's policy, you can leave, but you will leave all your contacts behind as well.
* Your information sources are under Facebook's control. Its algorithm builds your wall. You can choose your activities and follow groups, but you have little say in how much "other content" you see: ads, propaganda, fake news. I consider building my own sources of information a basic part of the right to be well informed, and I spend a lot of time finding good ones. Again, you can leave Facebook, but you will lose those sources.


> Is a choice from one option still a choice?

This is a fundamental question. Do you want to live in a golden cage with a golden path inside? Do you want to embrace the truth or live in a fable? I do not expect everyone to care, or even those who care to act. But I want to give those willing to listen everything I have learned along the way, and let them choose. I want them to have more than one option.

I work with data and with algorithms that analyze you, make predictions about you and bring benefits to everyone's lives. I know what it takes to act on this, and I can help. So why shouldn't I?


## What next ...

A few months ago I started looking for a better alternative to Facebook. It had to meet all of these requirements:

* simple: my friends and family must find the app simple and comfortable. My mum has to feel at ease with it, because there is no point in switching if nobody will message you ;-)
* messaging: it must send simple messages, images and video
* video conferencing: it must support video calls
* SSL: it must use SSL everywhere and support end-to-end encryption
* self-hosted: data must be stored on my servers, otherwise there is no point in leaving Facebook. Keeping your history away from internet vendors makes your communication private again, and I want to store it securely on my own servers.

I looked at several projects (ejabberd, Rocket, ...), but none met all the requirements until I found the Matrix project. What struck me at first sight was that it is not a client or a server but a protocol specification, and a federated one at its core. Then I realized that this is exactly what solves the monopoly problem.

The idea of federation is simple and resembles email. You may have your account at Gmail, yet you can still reach people on other networks, such as Yahoo, AOL, Seznam or your company's. No single network owns your identity, and it can be shared between them. If you want to change your email provider, you create a new account and send your contacts the new address, without losing any of them. It is that simple.

Matrix uses a similar concept, with a chat address for each user. Mine is:

> @peter:petercipov.com

Like an email address, it has a domain part (petercipov.com) and a user identifier (@peter). To contact someone, you just need their chat address.

Like email, Matrix is federated. You can use a hosting provider (matrix.org, for example, offers free chat hosting) or host it yourself, as I do.

Your data is stored on a homeserver, and when you contact someone on another homeserver, a copy of the message is transferred there, again just as with email.

You can also make voice and video calls, because Matrix supports WebRTC. You can call another device or even a web browser.

Matrix natively supports end-to-end encryption [1, 2], which can be enabled per room. Messages are then encrypted on the client before being sent to the server, so only their recipients can read them.

As a protocol, Matrix comes with a back-end reference implementation written in Python and several front-ends. My favourite is Riot.im. You can install it natively on iOS and Android, or use it as an Electron-based desktop app or a standalone single-page web app.

<figure>
  <img src="element.png" alt="Two phone screenshots of the chat app: a home screen with rooms and people, and a group conversation." >
  <figcaption>
    Element.IO user interface.
  </figcaption>
</figure>


## How
So I decided to host the chat service myself. My use case is purely personal. I am not doing it for money (I pay the bills :)), and not processing any data is the whole point of the project. My first step was to read the GDPR to find out what requirements I had to meet. I was happy to find the following [1](https://gdpr-info.eu/art-2-gdpr/) ...

> This Regulation does not apply to the processing of personal data: ... by a natural person in the course of a purely personal or household activity;

100% compliant :)

What is more, the people behind Matrix take this regulation seriously and will provide patches to make Matrix GDPR compliant [2](https://matrix.org/blog/2018/05/08/gdpr-compliance-in-matrix/). Commercial users should be covered within the next few patches.

## Self Hosting

I want everything simple and minimal to keep the cost/value ratio in my favour. I bought two small VPSes (virtual private servers) with 1GB of RAM each: one for the homeserver and one for the STUN/TURN server used for VoIP and video calls. The TURN server is pretty heavy on traffic and resources, hence the second VPS. You also need DNS records, in my case chat.petercipov.com for the Matrix homeserver and turn.petercipov.com for the STUN/TURN server.

All instances run the latest Debian and use public/private keys for access, with no passwords. All the processes I use are dockerized.

* https://hub.docker.com/r/petercipov/synapse-docker/tags/ for synapse homeservers
* https://hub.docker.com/r/petercipov/coturn-docker/ for STUN/TURN server
* https://hub.docker.com/r/_/postgres/ (alpine version) as data storage

You could use the official Synapse Docker image, which has everything wrapped inside and is much easier to start with. I prefer separate processes, so that I can scale and reshuffle instances myself.

Files and images are stored directly on the file system. I have set up size monitoring so that I can back them up or purge them when they reach the limit. There are other options, but for my purposes I am nowhere near that limit.

SSL is handled by Let's Encrypt and a certbot Docker image.
```bash 
docker run \
  -v `pwd`/data/ssl:/etc/letsencrypt \
  -e distinct=true \
  -e domains="domains....." \
  -e email="email...." \
  -p 80:80 \
  -p 443:443 \
  --rm pierreprinetti/certbot:latest
```

## Conclusion

Do not be fooled by the fallacy of free services.

Privacy is ultimately about ownership and control. It is an asset, like your finances, house or flat. You can hire a financial advisor or rent out a house, but you can also say no and end the arrangement. Before GDPR, you had little control over your privacy, and you were dragged into this business whether you wanted it or not. When you press "allow access to your mobile contacts", you send all your friends' data and create hidden accounts for them in advance, even for people who never signed up.

I see GDPR as a positive step towards giving us more power over our privacy. If you want to sell yours, that is your choice. I would rather play it safe, set my own rules and shape my life as I like. And yes, it is a lot of extra work, but believe me, it is worth it.
