---
title: "Traces - sane logging for asynchronous code"
date: 2015-12-17
description: "Normal logs make it hard to follow one request through async code. How traces group related messages so you can see what really happened."
featureAlt: "Painting of a glowing orange sunset over water and distant buildings."
summary: ""
tags: ["traces", "java"]
---

One problem you will face when building a distributed system is how to log properly, or more precisely, how to make your logs useful. The simple applications in tutorial blogs tend to skip logging altogether, yet logs are irreplaceable when you need to figure out why your system is failing. As a colleague of mine often says:

> There is never enough logging.

There are tons of frameworks to help you (slf4j, log4j, …), but in my opinion they deliver only half of the solution. Systems at scale produce huge numbers of log messages, across many services on many machines. All of them end up in one giant file or a set of giant files, and your job is to find a needle in that haystack. The task itself is trivial, but it takes longer and longer as the files grow.

You would save yourself a lot of trouble if related messages could be easily retrieved from those files, and that is the essence of tracing. You mark related log messages together and order them by occurrence. For example, you can group messages by request. Everything about database communication and your service logic, such as ACLs and custom business logic, then ends up in a single ordered sequence. It would be really handy if logs looked like this:

* API request started (path: /api/user, scope:all )
* Authorising session (session id: hgty7)
* Authorisation granted for reading
* User service is loading users
* Users loaded from database (count:452)
* Transforming to JSON
* Response sent successfully

You can trace messages, parameters and exceptions, which mean much more together than they do when logged separately.

Extracting this kind of value from ordinary logs takes processing, often an enormous amount of time. Sometimes the logs do not even contain enough information to group messages reliably. This is often the case in asynchronous programming, where a request is not processed by a single thread and so has no single stack trace. Instead, it is divided into a sequence of sub-tasks that run in order, but their actual execution time depends on the system scheduler and is non-deterministic. To be sure, you have to add extra information to each log message, such as some kind of id. At that point, you are already building custom traces.


## What is wrong with logging?
The core problem with logging is the loss of information and context. Inside the program you have the full context: you know where you are, what your inputs are and what comes next. You then write some of it to a file as text, often leaving out details you do not consider important.

Sometimes the cause of a failure is a single database exception or a null pointer, and one log message is enough. But what about semantic errors, when a bug is hidden in a complicated flow of messages between your services?

Distributed systems can also fail in other ways, such as network floods, and then you want to know the cause. It might be an error or a glitch in the code's semantics. Such errors are nearly impossible to reproduce, and by the time they occur, it is too late to add more logging. Your system is crumbling and you need to react as soon as possible.

Your logs end up stored somewhere, perhaps in Kibana or Graylog. These systems offer very limited search, without the joins we know from SQL databases, because such searches do not scale. We should therefore store already aggregated data in them, with a single input message combining all the messages from one request, configuration change, system start-up or tear-down. These aggregations are specific to each project, even though they share some similarities.

It is handy to log everything. That means all input parameters (or their sizes or ids, if they are large), and all computation steps in the business logic with their results. It also means all output actions, such as sending a message or responding to a request. All of this is then serialized as text and stored in the logging system.

## Example

Let's start with a single-node example: a remote API call that purges a user's history. Here is the pseudocode:

```java
public int purgeHistory(Trace trace, String userId) {  
  checkACL(trace, userId);
  purgeUserHistory(trace, userId);
  return respondOK(trace);
}
```

To purge a user's history, we first check the user's access rights, then purge the history and send back a success response.

```java
private void purgeUserHistory(Trace trace, String userId) {  
    Event purgingEvent = trace.start("Purging user history (userId)", userId);
     try {
      callRestService(trace, 1, "{\"data\": \"remove\"}", true);
    } catch (Exception ex) {
      trace.event("retrying purge call");
      .....
    }
     purgingEvent.end();
}

private void checkACL(Trace trace, String userId) {  
    ....
    trace.event("User has sufficient privileges (userid)", userId);
}

private int respondOK(Trace trace) {  
    ....
    trace.event("Responding OK ");
    return 200;
}

private String callRestService(final Trace trace, final int id, final String data, final boolean fail) throws Exception {  
    Event callEvent = trace.start("calling remote service (id, data)", id, data);
    String code;
     try {
      if (fail) {
        code = "500";
        throw new IllegalStateException("HTTP 500"); 
      } else {
        code = "200";
      }
    trace.event("call succeeded (response)", code);
    } catch (RuntimeException ex ) {
      trace.event("call failed", ex);
      throw ex;
    } finally {
      callEvent.end();
    }
     return code;
}
```

The Trace interface is very simple and contains two methods. You use `start` to record an event with a duration, which is marked as finished when you call its `end` method. You use `event` for the simpler case of something that happened at a single point in time.

```java
public interface Trace {  
    interface Event {
        void end();
    }

    Event start(String name, Object ... values);
    void event(String name, Object ... values);
}
```

<figure>
  <img src="example_trace.png" alt="JSON output of a trace: numbered events with thread names, timings and values, including a failed call with its stack trace and a successful retry." >
  <figcaption>
    Example of the trace.
  </figcaption>
</figure>


Every action in the code has been recorded. Each record has a timestamp (dt, or delta time, the number of milliseconds since the first event). It also has an id that is unique within the trace and grows with the execution order. It also contains the name of the thread (th) that executed the code, the message (n), values and exception stack traces. Some records refer to earlier ones (endOf) to note when a method finished.


## Amount of data
The downside is that all this produces a hell of a lot of data. This comes with traces, and you have to mitigate it with your own logic.

For high-frequency calls, you can sample logging by passing a NOOPTrace instead of a logging trace, recording only a percentage of calls. Even that portion of the data can be valuable, for example when looking for latency spikes. Alternatively, you can log traces only when an exception occurs. You lose real-time metrics about your system, but such a trace is still more useful than a plain stack trace. It really depends on what you need from your traces.

## Not a new idea at all
I did not invent this idea. It is simply handy and straightforward, and it has served me very well when trying to figure out "why it does not work".

It is the core idea behind Google's tracing system [Dapper](https://research.google/pubs/dapper-a-large-scale-distributed-systems-tracing-infrastructure/) and Twitter's [Zipkin](https://github.com/twitter/zipkin).

I did not use Zipkin because it focuses only on the duration of method calls, and I also wanted to log some of the data. It lacked this feature when I evaluated it and, to the best of my knowledge, still does. So I created a simple, stupid library that does what I wanted. Zipkin's logging via Scribe and processing via collectors is very sophisticated, and I do not want to compete with it. Its developers have done a great job, but for my environment it was overkill.

I used tracing in an application written with RxJava, mostly in a functional style. Its business logic consists of sequences of timers and waiting for responses from different services, which makes the code highly asynchronous. Before traces, when something failed, it was nearly impossible to reason about why. There was no complete flow, only a stack trace from an executor thread. And that is the core idea: a trace is your own custom stack with meaningful parameters. You decide what goes into it, and it accompanies a single request through its whole execution.

<figure>
  <img src="async.png" alt="Hand-drawn sequence diagram of IO, worker and timer threads passing watch, set, timeout and fire-watch messages." >
  <figcaption>
    Example of the flow.
  </figcaption>
</figure>

In my case, the service had 3 threads: one for IO, one for all event logic and one for timers.

A typical request watches a resource for changes. The request is registered and waits, not by blocking a thread but as a task in the timer executor with a timeout. If the timeout expires, the request is recovered and "no change" is returned. If a modification happens first, the timer is canceled and the watch returns "change occurred". I wanted to log this whole process: why the request was postponed, why it was recovered and which request did it.


## What is the difference?
My point is that Traces is much simpler than the other solutions. It assumes nothing up front and is as explicit as possible. It needs no annotations, thread locals, request contexts or meta-programming. There is simply no magic.

This may seem to contradict common approaches like aspect-oriented programming, where you write a set of selectors and the code is automatically instrumented. I would argue that no automation will give you a satisfying result, because you have to decide what is important at the level of individual lines. Automation also adds another level of abstraction to reason about. In my view, a meta-programming approach becomes unmaintainable, because code contains too many edge cases and they tend to spread over time.

Why not follow slf4j, with a statically accessible logger factory, so that you need no extra parameter in your methods? That works well for standard Java code, where the same thread runs all the methods and the context stays the same.

But when execution switches between threads, you lose your context, and worse, it may be "reused" by another request.

You could use some kind of token, but then you are back in the same situation, passing something through your methods to bind them together. So why not just pass a trace object? You then need no reflection magic, and the code does exactly what it says.

There is a simple rule for what to trace: anything that ends in a finite, preferably short, time. Infinite algorithms would produce infinite traces, and long-lived tasks would produce huge ones. You do not want to flood your memory with something endless, do you?

## Are traces distributed?
Simply put, yes and no :) You have to make them distributed.
<figure>
  <img src="distributed.png" alt="Hand-drawn diagram of a POST request going to a balancer node (1), then a processing node (2), then a cluster of Cassandra nodes (3)." >
  <figcaption>
    Distributed trace.
  </figcaption>
</figure>

Take a simple example with balancer nodes (B), processing nodes (P) and Cassandra nodes (C). When a request (1) arrives, the internal balancer logic routes it (2) to a processing node. The node processes it, calls Cassandra (3) and returns the response to the client.

To trace these calls, you generate a global trace id at the entry point and pass it through the layers. Each node logs a separate trace marked with that id, and afterwards you can load all the traces that share it. It is the same idea as passing trace objects through your code.

## Is it a "new blue"?
Traces are nothing new, just a different approach to logging. The question is whether you need them. If you have a distributed system with a complicated set of messages between services, I recommend considering them. You will have to work out how much data you can handle and modify your code accordingly.

That takes some work, but as I said, automation and general solutions based on aspects may not satisfy you. Even I am sometimes unsure at first what to trace, but when the code starts behaving strangely, I add more traces. I now use them everywhere, though that may not suit your case. As always, there is no silver bullet. Traces are simply a good aid when you are stuck with the same kinds of problems I have had.

You can find the sources [here](https://github.com/petercipov/traces).
