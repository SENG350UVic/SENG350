### Presentation Script - Node.js
**Problem 1 - Producer/consumer problem of handling shared resources; becomes difficult with too many threads**
Node.js is a JavaScript runtime environment that brings event-driven programming to web servers. To achieve this, a significant design decision was to build Node.js around an *event-driven architecture*. The main goal of this architecture is to allow Node to handle a large amount of requests asynchronously, through the use of an event-loop and a worker pool - a set of spawning background worker threads. Node's event loop is single-threaded, and accepts events triggered directly from the application level. Then, it queues and handles events synchronously. When an event needs to be operated on asynchronously at a later time, the event-loop delegates handling of this event to a new worker thread, that is dedicated to this event. When the event triggers, the background worker thread can then operate on it (via callback chains, etc.) and finally notify the main event-loop that the event is ready to be dispatched to its correct handler at the application level.

(Showing M3 C&C diagram for Node.js here)

The event-driven architecture allows Node to avoid the Producer/consumer problem, where contention for resources (events triggered, in this case) become a blocking point for the processing of a system. By avoiding threading in the main event-loop, Node achieves a more understandable flow of control where events pass in and out of the event-loop.


**Problem 2 - Various libraries in diff languages, across multiple protocols, must be used to create client-server implementations**
Node.js greatly leverages the V8 JavaScript runtime engine, and thus inherits many of its capabilities. By deciding to build on top of V8, Node.js becomes lightweight and performant, as the large majority of apps and functionality is built natively in JavaScript. Therefore, Node largely relies on JavaScript and avoids having to rely on too many external dependencies outside of the language it is primarily focused on.

Furthermore, Node's numerous battle-tested libraries can be easily attached on through its dependency package manager, npm. This integration allows Node to offer a large (And still growing) amount of resources to developers, who can tackle their development problems through the help of Node. With libraries built for Node.js, implementations - such as client-server models - can be easily realized by eager developers.
