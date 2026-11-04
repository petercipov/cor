---
title: "Comet: trendy and hairy"
date: 2013-12-27
description: "How websites send live updates, like chat messages, to your browser. A look at polling, long polling, streaming and WebSockets."
summary: ""
tags: ["web", "transport"]
---

Cometes is a Latin word with meaning "wearing long hair". Aristotle used it to describe stars with long hair: comets.

> Comet is a collection of techniques based on HTTP for processing long-living connections.

Let’s introduce a typical comet application - chat. As a client, you post messages and expect responses as other users respond - in real time. You want to see whether a user is typing a message and their status (online, offline).

Comet is responsible for transfer - from client to server and from server to clients. In most cases a single long-living connection is established to the server and when an event occurs a message is sent back and the connection is closed. The client then immediately establishes a new connection. This way it is possible to emulate client to client communication.

You can find comet inside many popular web applications.

* Gmail uses it for notifications, for chat.
* Facebook for everything :).
* Google Docs for its real-time multi-user editor.

## First attempts in 1996

Comet is an old technology. It was first introduced in 1996 with Netscape.
This was the first time when these two notions were introduced:

* server push - server is sending messages to clients
* client pull - client is registering for messages

Example request:
```
GET http://www.w3.org/pub/WWW/TheProject.html HTTP/1.1
```

Response:
```
Content-type: multipart/x-mixed-replace;boundary=ThisRandomString

--ThisRandomString
Content-type: text/plain

Data for the first object.

--ThisRandomString
Content-type: text/plain

Data for the second and last object.

--ThisRandomString--
```

The client pulls a request from the server for some image or HTML page. As the server pushes a response, the content of the DOM element is replaced with this new value (there can be delays between pushes).

Support in browsers is tragic, only Firefox supports it fully.

<figure>
  <img src="browser_support.png" alt="Logos of Firefox, Chrome, Safari, Opera and Internet Explorer, mostly greyed out." >
  <figcaption>
    Comet browser support.
  </figcaption>
</figure>

### Why was Comet not globally adopted?
Comet was introduced in 1996, but why was it not globally adopted?
The answer is simple: there was no demand. Let’s realize what kinds of web applications were modern at that time. Simple HTML pages with a few images, linked with hyperlinks. The main use case for comet was to show image advertisements - it is really only a tiny bit of its potential.

### Why is now a good time for real-time?
What has changed since 1996? What is the reason for you to consider whether it is time for change? Web applications have to show more and more real-time changes, show graphs, implement chat, browser notifications, client to client communication. On the other hand you do not want to refresh the whole page again and again. This way a page can become unusable. The second reason is that browsers now globally support more comet methods and comet is much easier to implement.


## Transport: Polling
<figure>
  <img src="polling.png" alt="Diagram of a client and a server over time: the client sends constant requests and the server answers each one straight away." >
  <figcaption>
    Polling as data transport.
  </figcaption>
</figure>

It is not a real comet technique. (It does not have "long hair" :) ). It simulates real-time interaction by flooding the server with a sequence of requests. The server immediately responds with a message or with no message. However, the simplicity of the technique is paid for with a lot of disadvantages:

* It does not scale - with more clients your server will lag. It is unusable even with a very low number of clients (you DDoS your own server ;) )
* It is ineffective - every request has HTTP overhead. In most cases there is no message, nothing is transferred except HTTP headers. In small scale it is negligible, but in large instances it is an issue you should consider.

## Transport: Long-Polling

<figure>
  <img src="long_poll.png" alt="Diagram of a client and a server over time: the client sends a request, the server answers later when it has a message, and the client then sends the next request." >
  <figcaption>
    Long polling as data transport.
  </figcaption>
</figure>
The difference between polling and long polling is the "long hair". Clients pull for messages and the server pushes only when some message occurs or the connection times out. This is a more efficient way of doing comet and it is the most widespread technique, because it works on every browser even though fancier methods are available.

The number of clients is problematic. When your server holds a thread for every request until the response, the number of clients is limited by the maximum number of threads your server can handle simultaneously. This is the well-known [C10K problem](https://en.wikipedia.org/wiki/C10k_problem). Fortunately, this problem was overcome recently in Java with the [Servlet 3.0 JSR 315](https://jcp.org/aboutJava/communityprocess/final/jsr315/) specification, where the request thread can be returned to the pool to handle other requests but the socket is left open and waiting for an event. The response is written from an event thread.

## Transport: Streaming

<figure>
  <img src="long_poll.png" alt="Diagram of a client and a server over time: the client sends a request, the server answers later when it has a message, and the client then sends the next request." >
  <figcaption>
    Streaming as data transport.
  </figcaption>
</figure>

It is like long-polling, but the server will push responses to the same connection multiple times. The connection is reused for more messages.
It can be done by creating an XHR request and registering a handler.

```javascript
xmlhttp.onreadystatechange = function() {

  if ( xmlhttp.readyState == 3) { //data has arrived
    //process chunk
  }
}
```

The server will respond with some data structure that can be deserialized. Data may be received in chunks. The program has to demultiplex and fire an event to the application when the whole message is received.

* This does not work in IE :( - IE does not return chunks, only the whole message when the connection is closed. This can be hacked, but it is trickier. You can bend IE by sending back JSONP - script tags with JavaScript and JavaScript can be executed on the fly in an iframe. You can read more [here](https://meteorserver.org/browser-techniques/) and [here](http://cometdaily.com/2007/11/18/ie-activexhtmlfile-transport-part-ii/).


## Transport: HTML 5 API
Previous techniques were mostly hacks. They bend browsers in a manner they were never meant to be used in. From the beginning HTTP was considered to be a one-way protocol and now we need to make it a two-way protocol. The preceding solutions work on all browsers but they do not work as we would like. HTML 5 comes to help. It brings a JavaScript API to make comet easier to implement.

### API: Event Sourcing
It is [standardized streaming](https://web.dev/articles/eventsource-basics). The client pulls and the server pushes messages to the same connection.
```javascript
if (!!window.EventSource) {
  var source = new EventSource('stream.php');
} else {
  // Resort to xhr polling :(
}

source.onmessage = function (event) {
  alert(event.data);
};

source.addEventListener('add', function(e) {
  if ( e.readyState == EventSource.CLOSED ) {
    // Connection was closed.
  }
}, false);
```
The browser handles the connection - when the connection is closed, the browser automatically reconnects.

### API: Websockets

<figure>
  <img src="websockets.png" alt="Diagram of a client and a server over time, with dotted arrows showing messages going both ways over one connection." >
  <figcaption>
    Websockets as data transport.
  </figcaption>
</figure>

After 20 years of web evolution we have finally realized that we need to rediscover what we already had in C for a long time - sockets. There is no extra magic. The client pulls for changes, the server parses headers and uses the TCP connection directly. This connection is bidirectional. It is a TCP socket for the web.

```javascript
var connection = new WebSocket('ws://go.to/stream');

connection.onopen = function () {
  connection.send('Ping'); // Send the message 'Ping' to the server
};

// Log errors
connection.onerror = function (error) {
  console.log('WebSocket Error ' + error);
};

// Log messages from the server
connection.onmessage = function (e) {
  console.log('Server: ' + e.data);
};
```


> RFC 2616: A single-user client should not maintain more than 2 connections with any server or proxy. 

Every browser has a limit on how many parallel connections it can establish. Therefore comet is often implemented with a single connection to the server and all messages are multiplexed over this single connection. If more than the limit are used, a request will simply wait until some connection is freed. This can bring unwanted lags.

<figure>
  <img src="parallel_connections.png" alt="Browser logos with their parallel connection limits: Firefox 6, Chrome 7, Safari 6, Opera 4, Internet Explorer 6." >
  <figcaption>
    Number of parallel connections.
  </figcaption>
</figure>


#### Server side
In the world of Java, comet was properly introduced by Servlet 3.0. It brings the possibility to do comet without hacks via a standard API.
```java
@WebServlet(
  name="myServlet", 
  urlPatterns={"/slowprocess"},
  asyncSupported=true)
public class MyServlet extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        //store context for later usage
        AsyncContext ctx = request.startAsync(); 
		...
        //you can register connection listener
        ctx.addListener(
        	new AsyncListener() {
            
              public void onComplete(AsyncEvent ae) {}
              public void onTimeout(AsyncEvent ae) {}
              public void onError(AsyncEvent ae) {}
              public void onStartAsync(AsyncEvent ae) {}
              
            }
        );
    }
}
```

The API is pretty simple. You mark the servlet to support long requests (asyncSupported=true). You create an AsyncContext from the request. You store it somewhere in memory. The doGet method is non-blocking. It finishes properly but the socket is still open. When an event occurs you pick up the context and write the response as for a usual request.

### Web applications are evolving.
From simple HTML pages to single page applications. Applications became more social, more user friendly, more useful. Comet is a part of the paradigm shift that brings web applications to a new level even though HTTP was not designed for it. A lot of people have realized it. The web is no longer a static set of CSS and HTML. It is a means to easily deliver apps to users written on a single platform. It does not matter whether you are using Linux, Windows, Mac, Android, iOS. You can run your apps on all of those systems, written once in JavaScript. Many have tried before but failed: Java Applets, Adobe Flash, Microsoft Silverlight.
To be honest, the current state of web programming is far from ideal. Making something cross-browser is a pain. Not all browsers support HTML 5 fully now, but all browsers make an effort to do so. This brings less hacking and more focus on business value.