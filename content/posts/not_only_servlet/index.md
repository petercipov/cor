---
title: "NO (not only) Servlet"
date: 2013-12-27
description: "A hard-to-find bug with Tomcat's async servlets corrupted our responses. What caused it, and why you may not need a servlet container at all."
featureAlt: "Painting of a small boat on hazy water at sunrise."
summary: ""
tags: ["web", "transport", "java", "servlet", "rewrite", "premature optimisation", "anti patterns", "assumptions"]
---

The main addition in the Java Servlet 3.0 specification was support for Comet techniques, which let you handle long requests without blocking. This was possible before, but only through container-specific APIs that tied you to a platform. Servlet 3.0 solved that with a standard API. The real issue now is the state of the containers themselves. I can only judge the two I have used heavily in the past: Tomcat and Jetty.

Servlet 3.0 was a big change. The synchronous request model suddenly changed, and the new behaviour had to fit into existing codebases without a global rewrite. The result was turmoil, as I will show using my own experience with Tomcat.

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

This is the typical structure of an asynchronous servlet, as found in every demo:

1. You make the request asynchronous by calling startAsync.
2. You register a listener for various state changes.
3. You set the request timeout.
4. You store the context in a global list, an executor or wherever your business logic requires. It acts as a kind of waiting list where requests wait for an event, such as a chat message that needs to be propagated to clients.
5. You avoid long operations such as waiting and simply return. The method ends and the thread goes back to the pool, but the socket stays open.

It looks easy, even simplistic, but it hides some weak points that are not visible at first sight.

I was in charge of implementing a [Pub-Sub](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) feature in a product. One part of the application was a JSON-RPC-like API using a thread-per-request model, and the other was a notifier.

The Servlet 3.0 API is well documented, so the first implementation came together quickly. Then, out of the blue, responses became corrupted. Sometimes a JSON response would contain a tiny part of another JSON document or a fragment of an image. I spent a lot of time debugging without any result. To make matters worse, it happened in production at a level of traffic I could not reproduce locally. In the end I wrote to the Tomcat forums and received this reply:

    Did I miss some critical lesson about flushing async responses? 
    HttpServletRequest and HttpServletResponse objects 
    a) are not thread-safe 
    b) are cached and reused for subsequent incoming requests. 

In fact, Tomcat (and [Jetty as well](http://jetty.4.x6.nabble.com/Jetty-request-recycling-td4959197.html)) recycles request and response objects, replacing only their internals, to avoid unnecessary garbage collection. The downside is that this breaks the isolation between requests, and tons of [unpredictable bugs](https://wiki.apache.org/tomcat/FAQ/KnownIssues#ImageIOIssues) can follow. Life without isolation is really painful: a bug in one place can non-deterministically corrupt other places that are perfectly fine.

For someone implementing a feature, step 4 is where isolation problems appear. If you forget to clean up the structure properly, you have a bug. That happens, but finding it is a pain because it is non-deterministic. In my view, servlet containers are not safe enough. Multithreaded code should use as many immutable objects as possible, which makes it simpler and more readable. And containers certainly should NOT recycle requests unless you explicitly ask them to.

Out of frustration, I started digging into the source code, hoping to debug it and, with luck, patch it. No such luck. After years of maintenance without a rewrite, the sources are overcomplicated and full of fuzzy bits like [this](http://grepcode.com/file/repository.springsource.com/org.apache.coyote/com.springsource.org.apache.coyote/7.0.26/org/apache/tomcat/util/net/NioEndpoint.java#1205):

```java
 sk.attach(attachment);//cant remember why this is here
```

The codebases are improving over time. Jetty was completely rewritten in version 9, so places like this are less common, and you will not find any "can't remember why" comments in Tomcat 8's NIO connector. The main problem, however, remains in both containers: request recycling.

All of this brings me to the ultimate question:

## Do you really need servlet container?
Do you need all the fancy servlet features, like hot deployment, WAR class-loader isolation and life cycle management? Or do you just need an HTTP service? Tomcat and Jetty are widely used, but do you need all their power and abstraction? They were designed to meet many different demands, and I am simply asking whether those demands are yours. If not, why not use plain old embedded HTTP? Why use servlets at all?

There are many other stable options written from scratch for non-blocking communication, such as Netty and Vert.x.

If you are free to choose your technology, I recommend avoiding a servlet container for comet applications until you really need one. You should be able to clearly explain why it helps. If you cannot avoid it, be prepared for many problems caused by leaky abstractions and caching. And as we know, caching is hard.
