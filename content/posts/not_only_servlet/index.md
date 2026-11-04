---
title: "NO (not only) Servlet"
date: 2013-12-27
description: "A hard-to-find bug with Tomcat's async servlets corrupted our responses. What caused it, and why you may not need a servlet container at all."
summary: ""
tags: ["web", "transport", "java", "servlet", "rewrite", "premature optimisation", "anti patterns", "assumptions"]
---

The main addition of Java Servlet 3.0 spec was the introduction of Comet techniques. You can handle long requests without blocking. You could have done this before but you had to become platform specific. You depended on a servlet container specific API. Servlet 3.0 brought a standard API, so this is no issue anymore. What is the issue is the state of those containers. I can only judge two of them that I have been heavily using in the past. To be more precise, Tomcat and Jetty.
Servlet 3.0 was a big change. Now out of the blue the synchronous request model was changed and the Servlet 3.0 change had to be transferred to the new code-base without a global rewrite of the current code. The result was turmoil. I will demonstrate it on my case with Tomcat.

```java
public class AsyncLongRunningServlet extends HttpServlet {

protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

	//(1) create async context
    AsyncContext asyncCtx = request.startAsync();
    //(2) register listener
    asyncCtx.addListener(new AsyncListener() {
       
        ... onComplete(AsyncEvent asyncEvent) ....
            
        ... onError(AsyncEvent asyncEvent)...
            
    	... onStartAsync(AsyncEvent asyncEvent)...
 
    	... onTimeout(AsyncEvent asyncEvent)...
    });
    //(3)set timeout
    asyncCtx.setTimeout(9000);
 
    //(4) store it somewhere in the heap for your purposes
    ....
    //(5) return from method - do not block this thread till response
    return;
}
 
}
```

This is a typical scheme of an asynchronous servlet, you can find it in every demo example:

1. You make the request asynchronous by calling startAsync
2. You register a listener for various state changes.
3. You set the request timeout
4. You store it to some global list, executor, whatever your business logic requires. This is some kind of waiting list, where requests wait for some event (e.g.: a message was sent and it is propagated to clients in case of chat application)
5. You should not make any long operations here like waiting etc. You just return. The method ends, the thread is returned to the pool but the socket is still open.

It seems easy, simplistic, but it hides some weak points that are not visible at first sight.

I was in charge of implementing a [Pub-Sub](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) feature in a product. So part of the application was a JSON-RPC like API, with a thread per request model, and the second one was a notifier. Servlet 3.0 API is well documented, so the first implementation was done quickly. But out of the blue responses became corrupted. Sometimes you got a response that contained a JSON response with a tiny part of some other JSON or a part of image. I spent a lot of time debugging with no result. This mistake was unfortunately occurring in production with some amount of traffic that I could not locally reproduce. In the end I wrote to Tomcat forums and I got this response:

    Did I miss some critical lesson about flushing async responses? 
    HttpServletRequest and HttpServletResponse objects 
    a) are not thread-safe 
    b) are cached and reused for subsequent incoming requests. 

In fact, Tomcat (and [Jetty as well](http://jetty.4.x6.nabble.com/Jetty-request-recycling-td4959197.html)) recycle request and response objects. They replace only internals. This is done to avoid unnecessary garbage collection. But on the other hand this breaks the isolation of requests and tons of [unpredictable bugs](https://wiki.apache.org/tomcat/FAQ/KnownIssues#ImageIOIssues) can occur. Life without isolation is really painful, because if you make a bug in one place it corrupts non-deterministically other places that are fine.

From a feature implementer's point of view, point 4 brings problems with isolation. You can forget to properly clean the structure. Yes, you make a bug. It happens, but finding it is a pain - it is non-deterministic. From my point of view servlet containers are not safe enough. All code that is used for multithreaded programming should have as much immutables as possible. It makes code more readable and simpler. And certainly they should NOT recycle requests without your direct intent.

So out of frustration I started to dig into the source code. I wanted to debug it and with luck patch it. No luck here. Sources are overcomplicated due to long maintenance without rewrite and they are full of fuzzy bits like [this](http://grepcode.com/file/repository.springsource.com/org.apache.coyote/com.springsource.org.apache.coyote/7.0.26/org/apache/tomcat/util/net/NioEndpoint.java#1205):

```java
 sk.attach(attachment);//cant remember why this is here
```

The codebase is improving over time. Jetty was completely rewritten in version 9, so places like this are less common. You do not find "can't remember why" places in Tomcat 8 NIO connector implementation. But the main problem, the request recycling is still present in both web containers.

This all brings me to the ultimate question:

## Do you really need servlet container?
Do you need all the fancy servlet features like hot deploy, WAR class-loader isolation, life cycle? Is it not your case, just an HTTP service? Tomcat / Jetty are heavily used, but do you need all of their power and abstraction? They were precisely designed to fulfill multiple demands. I am just asking whether these cases are yours. If not, why not just use "plain old embedded HTTP"? Why use servlets at all?

There are a bunch of other possibilities that were written from scratch for non-blocking communication and they are stable (Netty or Vert.x).

If you can choose technology I would recommend you to avoid a Servlet container for comet applications until you really need one (you can clearly answer the question: why is it actually helpful?). If you cannot, be prepared to solve tons of problems of too blurry abstraction and caching. And as we know caching is hard.