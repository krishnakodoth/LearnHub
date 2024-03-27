# How to add realtime updates to our React application

[https://ably.com/blog/react-realtime-updates](https://ably.com/blog/react-realtime-updates)

Imagine having to restart WhatsApp whenever you anticipate a new message, or needing to reload the page every time you expect an update on the big game. That would be a terrible user experience!

Today, users expect the UI to automatically update the moment information becomes available from the backend, so, of course, we must enable them in our applications.

There are a handful of technologies, techniques, and services you can use to implement live updates in React.

Whether you’re building chat, realtime notifications, multiplayer collaboration features, or some other type of live update, you’re sure to find a method that works for you here.

## To poll, or to push, that is the question
Realtime update methods fall into two categories - polling or pushing.

Polling involves querying the server for updates on the off chance there’s new information available.

Pushing, on the other hand, provides a way for the server to push fresh information to the client as soon as it’s ready.

Polling incurs more latency than pushing and is less network efficient, but since polling is based on HTTP, which you’re using anyway, it’s straightforward to implement and universally compatible.

As you will see in this post, pushing conserves resources and favors lower latency.  At the same time, it requires a custom technology like [WebSockets](https://ably.com/topic/websockets) or [Server-Sent Events](https://ably.com/topic/server-sent-events) that might require some new reading.

## Short polling with fetch

One of the simplest ways to achieve live updates in React is to have the client send a background fetch request at an interval.

If new data is available on the server, then it is returned in the response. Otherwise, the response is empty.

[Polling](https://ably.com/blog/google-polling-like-its-the-90s) is simple to implement but is highly inefficient.

* Say you make a request every 60 seconds. If an update becomes available on the server moments after the client check, the update will have to wait until the next client check (60 seconds).
* Polling at a frequent interval may translate into frequent updates, but will result in unnecessary traffic and high overhead, depending on the frequency of updates.
* Every request incurs around 800 bytes of overhead from the HTTP headers (without cookies) and is briefly delayed while the underlying connection is established, potentially causing a noticeable latency for your users.

Recommendation: While simple to implement, short polling is highly inefficient for most situations. It's only occasionally a good fit when polling intervals are long, new information arrives at a predictable rate, and you want to keep the code as simple as possible - even though you accept that you will run into reliability problems in production. It’s usually not a good option, but I’m including it here to frame the other methods.

## Long polling with fetch

[Long polling](https://ably.com/topic/long-polling) is an optimized version of short polling whereby, instead of returning an empty response when no update is available, the server keeps the HTTP connection idle until there’s an update to share.

By holding the connection open until an update is available, the server can send fresh data to the client immediately as soon as it’s ready.

As a result, there are no empty checks, which reduces the number of asynchronous requests and the overall overhead of polling.

This method, also known as [Comet](https://infrequently.org/2006/03/comet-low-latency-data-for-the-browser/), is more efficient than short polling, but it’s far from perfect.

* While the server doesn’t have to burden unnecessary checks anymore, it now has to keep multiple connections open indefinitely, which ties up server resources like memory.
* This accelerates the need for additional servers, but because HTTP is stateless by design, spreading the stateful idle connections across multiple hosts will require a unique approach to load balancing.
* Neither short polling nor long polling provide any guarantees around message ordering, they don’t handle automatic reconnections with message continuity either.

Recommendation: Long polling is inherently more efficient than short polling and it’s not much more work to implement, so you may as well use it. At the same time, long polling is resource-hungry and, like its predecessor, short polling, won’t achieve the lowest possible latency because of the overhead each message incurs.

## Server-Sent Events

[Server-Sent Events (SSE)](https://ably.com/topic/server-sent-events) is a browser API and protocol that enables the server to push fresh information to the client as soon as it’s available.

Under the hood, SSE opens a long-lived HTTP connection, however, unlike long polling, where each message creates a new connection, SSE reuses the same connection, reducing overhead and, in turn, latency.

The [SSE API](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) is available in every modern [browser](https://caniuse.com/?search=eventsource), and it features some nice-to-have capabilities you’d ordinarily have to implement yourself with polling, including automatic reconnections. Working in tandem with [event stream protocol](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#event_stream_format), SSE can resume the message stream from the last known message, enabling a seamless realtime experience, even if the client temporarily disconnects.

Compared to long polling, SSE is more efficient and has automatic reconnections built-in. Does that make it the perfect solution? Like any method to achieve live updates in React, there are some considerations you should be aware of as well:

* With SSE, messages flow one-way from the server to client over the same connection. This is fine for live updates such as [notifications](https://ably.com/blog/how-to-add-notifications-to-your-react-app) or [new chart data](https://ably.com/blog/how-to-use-google-charts-with-react), but it might not be the most appropriate option for games or chat where the information has to flow bidirectionally over the same channel.
* Similar to long polling, every client opens a long-lived connection to the server, tying up resources like ephemeral ports, memory, and bandwidth.
* As your application scales, you’ll need to beef up your server or spread incoming connections across multiple hosts. However, because the connections are stateful, this will require a unique approach compared to ordinary HTTP requests.
* Neither polling nor SSE support online presence, making it tricky to show who’s online in a chat application or team collaboration app, for example.

Recommendation: SSE is a superior option to polling, and is especially well-suited for one-way updates such as notifications or [realtime feeds](https://ably.com/tutorials/newsfeed-react). The API benefits from widespread browser support, and even includes handy features you’d otherwise have to implement yourself. The downside is that information can only flow over the connection one-way from the server to client, which might not be appropriate for things like multiplayer collaboration, chat, or games, where the information has to flow both ways, potentially in parallel.  While you could send updates from the client to the server over a separate channel, that introduces complexity on the backend to link incoming messages to their outgoing connection.

## WebSockets
[WebSockets](https://ably.com/topic/websockets) are a communication protocol that enable bidirectional communication between applications.

They are a great choice when two-way communication is needed such as chat and multiplayer collaboration.

Additionally, WebSockets are well-suited when you need to push fresh data from the server as soon as it’s available - like live sports score updates, updates on the location of a package delivery, or perhaps realtime chart data.

Because they are [full-duplex](https://www.techtarget.com/searchnetworking/definition/full-duplex), information can flow in both directions simultaneously, making WebSockets an attractive option for high throughput scenarios like an online multiplayer game as well.

* Compared to [polling](https://ably.com/blog/websockets-vs-long-polling) or [SSE](https://ably.com/blog/websockets-vs-sse), which inherit the benefits of HTTP, WebSockets are a totally custom protocol. They won’t work with your existing HTTP proxy since, well, WebSockets aren’t HTTP, meaning you’ll need to implement things like compression yourself. In general, there’s normally quite a bit of work to do on top to achieve reliable realtime updates in production.
* Although WebSockets benefit from [widespread browser support](https://caniuse.com/?search=websocket), it’s sometimes necessary to have fallback options like SSE or long polling, in environments where WebSockets aren’t supported, such as a hospital with a stringent firewall. This could involve double the work to implement, and double the complexity to scale, as the approach to each is fundamentally different.
* WebSockets ensure messages are delivered in the order they were sent, which is generally considered a plus. Except, this also means WebSockets are susceptible to [head-of-line (HOL)](https://en.wikipedia.org/wiki/Head-of-line_blocking) blocking. This is a deliberate trade-off that works well for most projects, but there may be scenarios where you want to achieve the highest possible throughput and lowest latency, even though it means messages sometimes arrive out of order (or not at all).

Recommendation: WebSockets are purpose-built for realtime updates, and they’re remarkably versatile. Whether you need unidirectional updates, bidirectional updates, or both, WebSockets are a futureproof choice. The downside is that you’ll need to spend more time than you’d probably like reinventing features you got for free with HTTP like compression and error handling. Not unlike long polling or SSE, their stateful nature makes them [challenging to scale](https://ably.com/topic/the-challenge-of-scaling-websockets).

If you’re tempted by WebSockets, check out my [complete guide to WebSockets with React](https://ably.com/blog/websockets-react-tutorial) in which I share not only how to use WebSockets, but how to best manage the connection lifecycle in React.

A realtime platform like Ably

As you’ve seen throughout this post, pushing messages from the server to the client is the superior option to polling, but it comes with some challenges.

* Every push method we’ve looked at requires the client and server maintain a long-lived connection which ties up resources, and complicates horizontal scaling, as you now need a way to synchronize state between backend servers.
* If the client disconnects, you need a way to efficiently reconnect and, ideally, replay all the messages they missed while offline.
* I haven’t mentioned this yet - but you’ll need to implement your own pattern to route messages where they need to go - for example, to every client (broadcast), a specific client (one-to-one), or a group of specific clients (one-to-many).

The methods we’ve looked at so far tick some of the boxes, but not others. For example, SSE handles reconnections automatically, but it’s one-way only and incurs more latency than WebSockets. WebSockets support bidirectional communication with the lowest latency, but they require a lot of development work to implement, and are challenging to scale. Both lack a well-thought-out messaging pattern to route messages to specific subscribers in a clean and orderly way.