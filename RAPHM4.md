# Milestone 4: Reverse Engineering Design Patterns

Name: Raphael Bassot
Course: SENG350 - Software Architecture
Group: #2

## Part 1: Reference Design Patterns

### Twisted Design Pattern: The Reactor Pattern

To achieve their goal of creating an event-driven engine, Twisted takes advantage of the **Reactor** design pattern. This pattern falls under the _Concurrency patterns_ type and is outside of the Gang of Four design patterns. The Reactor pattern decouples the event source with the event handlers and does not let them interact; this results in no subscribing, communicating or updating between consumer and producer. Instead, a central service handler acts as an intermediary and directly dispatches events from their sources to their event handlers.

**The Problem:** In Twisted middleware, the event-driven architecture depends on how triggered events, upon completion of a task, are handled within the system. In this architecture, Events are 'consumables' that must move from producer to consumer for the system to react to a change adequately. From the source, Events wait until the handler is available to service them; when available, the event handler invokes a chained callback of functions that perform the desired work in response to the event trigger. The network between event sources and handlers is complex. Multiple Event Sources send Events concurrently during the runtime of an application. The correct handler must adequately service these Events as soon as they are accessible. Additionally, should one event be in the process of being handled, the system shall not block on other events, pre-emptions made accordingly. The harmonization of these tasks is a significant problem that Twisted must manage.

**The Solution:** Twisted solves this problem by leveraging the Reactor design pattern. This design pattern separates Twisted into three parts - event sources, the service handler (or reactor event loop), and the event handlers. The reactor is the means of controlled communication between the event sources (such as a user input click) and event handlers (such as loading a new page). Twisted takes advantage of the event loop in this pattern to handle the flooding of concurrent events coming from the sources. The reactor event loop receives from the source, and has knowledge on the state of the ["network, file system, and timer events"](http://aosabook.org/en/twisted.html) for the system. Using this, the reactor can dispatch the events to the correct handlers at the next moment where the handler can start its synchronous operation on the event.

Twisted implements this pattern in their `twisted.internet` package, where developers can handle events efficiently in their single-threaded environment. An example is shown below, where the Twisted Reactor could be used is in an asynchronous URL fetcher. The reactor starts an event loop and receives any generated events (from the getPage() function) to then handle and dispatch them to the respective handlers (like the processPage() function). This example is taken from: "http://aosabook.org/en/twisted.html"

```javascript
    from twisted.internet import reactor
    import getPage

    def processPage(page):
        print page
        finishProcessing()

    def logError(error):
        print error
        finishProcessing()

    def finishProcessing(value):
        print "Shutting down..."
        reactor.stop()

    url = "http://google.com"
    # getPage takes: url,
    #    success callback, error callback
    getPage(url, processPage, logError)

    reactor.run()
```

**The Participants:** Participants in Twisted's Reactor pattern are the three groups of elements previously mentioned: functions that trigger the events (event sources), the intermediate reactor service handler, and the functions that act as event handlers for a specific event type. Event sources have the purpose of generating a trigger on the completion of an event, such that the system can have the chance to react appropriately to it. The reactor has multiple tasks including: blocking any individual events received until the correct handler is ready to service them (also known as demultiplexing), understanding the system's state to be able to determine dispatch times and destinations, and dispatching the correct events to the respective handlers that deal with them. Lastly, the event handlers' purpose is to accept the incoming event from the reactor, and to operate on the event in a way that allows the system to react to the event that was received. Usually, handlers process events synchronously at their next available timeslot, and use callback functions to follow the logical flow of the program in response to the received event.

There are constraints with regard to the three participant groups. Most importantly, the Reactor pattern does not allow communication or observation of state between any event source and event handler. Events have to pass through the reactor event loop instead, which further decouples the producers and consumers, and allows them to focus on smaller tasks, only worrying about a subset of the work done throughout the lifetime of an event in the system. This idea increases modularity, but in turn can become harder to debug for developers, because the flow of control is more obscure; it can be difficult to identify which component has control over the event at a given point in time.

Multiple reactor implementation for various purposes can be found in the `twisted.internet` module. The `reactor.py` file allows developers to install a reactor implementation. These implementations are found in the same directory and each offer a slightly different flavour of a Reactor-based pattern. They can be seen in the following files: `aynscioreactor.py`, `gireactor.py`, `pollreactor.py`,`wxreactor.py`, `kqreactor.py`.

**The Consequences:** The benefits brought to Twisted from applying the Reactor pattern is that programming in an event-driven environment becomes more modular. The three components required to process and handle events become separated to focus on the scope of their respective tasks, without worrying about the state of the other components in the model. Additionally, the pattern acts as an abstraction between the running application and the reactor implementation beneath it. The application can redirect events away from itself, to be handled by the components of the reactor model, and can guarantee that at some point it will receive instruction on how it should behave in response to that event.

The main disadvantage with this design pattern is that it can be hard to understand the flow of control for an event, and in turn difficult to debug. This is because the pattern decouples each reactor-based component from one another, meaning that the event can be in the process of being operated on in any of the involved components.

## Part 2: Addtional System Design Patterns

### Node.js Design Pattern: The Reactor Pattern

**The Problem:** Node.js, like Twisted, needs to handle the receiving, processing, and result-returning of multiple events that enter the system. Incoming events should be blocked from progress until a respective handler can process them, but they should not block the processing of other events or preempt operations in any way. Additionally, a fully multi-threaded environment should be avoided, due to the complexity of the control flow of these events, and the drawbacks on performance towards asynchronous programming that Node.js clearly focuses on.

**The Solution:** The Node.js runtime relies heavily on a slightly different approach to the **Reactor** design pattern, similar to Twisted's pattern. The idea follows the basic organization of the pattern, with a main event loop that picks events from the queue, and dispatches events from event source to components that can operate on events. Node varies in design by its use of a _worker pool_ - a group of multiple threads that synchronously perform tasks given to them by the event loop. The event loop is single-threaded, which is ideal for asynchronous programming, in that it benefits from better performance and is less complex than working with multiple threads at all times. Upon receiving a new event, the event loop redirects the event to a worker thread that can operate on the event and produce a desired result, while the event loop continues it job of receiving and dispatching. Once a worker thread completes it work, the result is returned to the event loop, who operates on the specified callback function with this result at its next availability. This callback represents the event handler that changes the state of the higher-level application. Ultimately, Node.js controls the entire flow of an event through the system by introducing a worker pool that operates synchronously and completes the processing step of every event internally before returning to the application.

**The Participants:** The elements in this Reactor pattern implementation are: the event queue, the main event loop, and the worker pool. The event queue is where events arrive from their application-level sources, and wait until the event loop can handle them. The event loop receives the events and dispatches them at an appropriate time to a thread in the worker pool. Each worker thread processes a result for the event, which the event loop can then apply to the respective callback to obtain a desired instruction on the application-level behaviour around that event. These three sets of components are constrained in their purpose and who they can communicate with. The event queue (and event sources) are unaware of the state of any thread in the worker pool, and rely on the event loop to behave as an intermediary between them. Additionally, worker threads are constrained by the event loop directly in that they can only process on an event directly dispatched to them by the loop, and perform no work otherwise.

**The Consequences:** Node.js benefits from this modification to the Reactor design pattern that was seen in Twisted. A significant benefit from the use of worker threads is the increased performance in processing events. Worker threads work synchronously - therefore they only focus on one task at a time - and many of these threads can parallelize the amount of work that Node performs. This contrasts Twisted's approach, which would rely on the developer to create separate event handlers in multiple threads to achieve the same result. Node's automatic spawning of these worker threads greatly improves its efficiency over the Twisted implementation. Compare to Twisted, Node.js does suffer from a dependency on the JavaScript language. While JS is efficient for web applications, it may not provide the flexibility that Twisted has, which largely relies on Python and can be further extended to other languages.

### Flask Design Pattern: Plugin-based Reactor Pattern

**The Problem:** First off, Flask is a microservice framework for Python and thus does not natively follow the Reactor-based pattern, nor is it built around asynchronous event-driven programming that leverages an event loop. Flask is largely built around the idea of being minimalist and providing developers with a flexible framework to achieve desired outcomes. Therefore, it is possible to recreate a reactor-based environment in Python Flask, despite the fact that Flask is not built around this pattern. The problem here will be described as a scenario where a developer wants to create an event-driven environment similar to those implemented in Node.js and Twisted. Event sources and event handlers need to be decoupled, and provided with the proper resources to complete their tasks. Additionally, the system shall not block while a single event is being handled, such that the program can efficiently handle multiple events at once.

**The Solution:** To achieve the Reactor design pattern, libraries are available as plugins to Flask. One useful plugin for this purpose is to use Gevent - a ["coroutine-based Python networking library that uses greenlet to provide a high-level synchronous API on top of the libev/libuv event loop"](http://www.gevent.org/). Gevent is built around the idea of using an event loop to handle events that come from the operating system. The event loop uses polling register for an event, to be able to determine when an event arrives. Once the event arrives, the event loop then reacts to them by running greenlets - thread-like structures that perform the processing of an event. Essentially, the event loop handles the dispatching of events to the event handlers, which are greenlets in Gevent. With this support developers can create a non-blocking, asynchronous environment that can comfortably process events as they come in.

**The Participants:** Flask Gevent involves event sources, an event loop, and the event handlers. Event sources are typically the operating system, or any application-specific operations that trigger events upon a user I/O input. The event loop runs continually in the Gevent environment, and the event handlers are the greenlets. Greenlets only receive event through the event loop, and do not communicate with the operating system or application level components.

**The Consequences:** Flask Gevent benefits Python developers because it is a simple yet robust solution to creating an asynchronous environment that can handle multiple events. However, efficiency is in question because Flaks does not natively support the Reactor pattern, as a Gevent library is required to achieve the desired behaviour.
