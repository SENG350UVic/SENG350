# Milestone 4: Reverse Engineering Design Patterns

Name: Raphael Bassot
Course: SENG350 - Software Architecture
Group: #2

## Part 1: Reference Design Patterns

### Twisted Design Pattern: The Reactor Pattern

To achieve their goal of creating an event-driven engine, Twisted takes advantage of the **Reactor** design pattern. This pattern falls under the *Concurrency patterns* type and is outside of the Gang of Four design patterns. The Reactor pattern decouples the event source with the event handlers and does not let them interact; this results in no subscribing, communicating or updating between consumer and producer. Instead, a central service handler acts as an intermediary and directly dispatches events from their sources to their event handlers.

**The Problem:** In Twisted middleware, the event-driven architecture depends on how triggered events, upon completion of a task, are handled within the system. In this architecture, Events are 'consumables' that must move from producer to consumer for the system to react to a change adequately. From the source, Events wait until the handler is available to service them; when available, the event handler invokes a chained callback of functions that perform the desired work in response to the event trigger. The network between event sources and handlers is complex. Multiple Event Sources send Events concurrently during the runtime of an application. The correct handler must adequately service these Events as soon as they are accessible. Additionally, should one event be in the process of being handled, the system shall not block on other events, pre-emptions made accordingly. The harmonization of these tasks is a significant problem that Twisted must manage.

**The Solution:** Twisted solves this problem by leveraging the Reactor design pattern. This design pattern separates Twisted into three parts, event sources, the service handler (reactor event loop), and the event handlers. The Reactor controls communication between the event sources (such as a user input click) and event handlers (such as loading a new page). Twisted takes advantage of the event loop in this pattern to handle the flooding of concurrent events coming from the sources. The Reactor Event Loop knows the state of the ["network, file system, and timer events"](http://aosabook.org/en/twisted.html) for the system. Using this information, upon receiving, the Reactor can dispatch the Events to the correct handlers at the next free moment. Then the handler can begin its synchronous operation on the Event.

TThis pattern appears in the `twisted.internet` package, where developers can handle events efficiently in their single-threaded environment. The javascript source code shows this pattern in the example below, where the Twisted Reactor uses an asynchronous URL fetcher. The reactor starts an event loop and receives generated events from the `getPage()` function. Once received, the reactor handles and dispatches them to the respective handlers (like the `processPage()` function). This example is from the [Twisted]( http://aosabook.org/en/twisted.html) page of *The Architecture of Open Source Applications* textbook.

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

**The Participants:** Twisted's Reactor pattern participants are the three groups of elements previously mentioned: functions that trigger the events (event sources), the intermediate reactor service handler, and the functions that act as event handlers for a specific event type. Event sources have the purpose of generating a trigger on the completion of an event, such that the system can have the chance to react appropriately. The Reactor participant has multiple tasks. The first task is to include and block any individual events received until the correct handler is ready to service them (also known as demultiplexing). Next, understand the system's state to determine dispatch times and destinations, and lastly, dispatching the correct events to the respective handlers that deal with them. The last participant, the Event handlers, is in place to accept the incoming event from the reactor, then operate on the said event. Usually, handlers process events synchronously at their next available timeslot and use callback functions to follow the logical flow of the program in response to the received event.

There are constraints concerning the three participant groups. Most importantly, the Reactor pattern does not allow communication or observation of state between an event source and event handler. Events have to pass through the reactor event loop instead, which further dissociates the producers and consumers and allows them to focus on smaller tasks, only worrying about a subset of the work done throughout the lifetime of an event in the system. This idea increases modularity; however, it makes debugging harder for developers because the control flow is obscure. It can be challenging to identify which component has control over the event at a given point in time.

The `twisted.internet` module contains multiple reactor implementations for various purposes. The `reactor.py` file allows developers to install a reactor implementation. These implementations are in the same directory, and each file offers a slightly different flavour of a Reactor-based pattern. The following files: `aynscioreactor.py`, `gireactor.py`, `pollreactor.py`,`wxreactor.py`, `kqreactor.py` contains these pattern implementations.

**The Consequences:** The benefit brought to Twisted from applying the Reactor pattern is that programming in an event-driven environment becomes modular. The three components required to process and handle events become separated to focus on the scope of their respective tasks without worrying about the state of the other components in the model. Additionally, the pattern acts as an abstraction between the running application and the reactor implementation beneath it. The application can redirect events away from itself to be handled by the components of the reactor model. It can guarantee that at some point, it will receive instruction on how it should behave in response to that event.

The main disadvantage with this design pattern is that it can be hard to understand the flow of control for an event, making debugging difficult. Decoupling each reactor-based component from one another causes this disadvantage, as an Event could be nondeterministically in any of the active components.

## Part 2: Addtional System Design Patterns

### Node.js Design Pattern: The Reactor Pattern

**The Problem:** Node.js, like Twisted, needs to handle the receiving, processing, and result-returning of multiple events that enter the system. Incoming events should be blocked from progress until a respective handler can process them, but they should not block the processing of other events or preempt operations in any way. Additionally, Node.js avoids a fully multi-threaded environment due to the complexity of the control flow of these events and the drawbacks on performance towards asynchronous programming.

**The Solution:** The Node.js runtime relies heavily on a slightly different approach to the **Reactor** design pattern than the Twisted one. The idea follows the essential organization of the pattern, with the main event loop that picks events from the queue and dispatches them from the event source to handlers. Node varies in design by using a *worker pool*, a group of multiple threads that synchronously perform tasks given to them by the event loop. The event loop is single-threaded, which is ideal for asynchronous programming in that it benefits from better performance and is less complex than working with multiple threads. Upon receiving a new event, the event loop redirects the event to a worker thread that can operate and produce the desired result. The event loop can now freely continue its job of receiving and dispatching. Once a worker thread completes it, the result is returned to the event loop, who operates on the specified callback function at its next availability. This callback represents the event handler that changes the state of the higher-level application. Ultimately, Node.js controls the entire flow of an event through the system by introducing a *worker pool* that operates synchronously. This addition completes the processing step of every event internally before returning to the application.

**The Participants:** The elements in this Reactor pattern implementation are the event queue, the main event loop, and the worker pool. The event queue is where events arrive from their application-level sources and wait until the event loop can handle them. The event loop receives the events and dispatches them appropriately to a thread in the worker pool. Each worker thread processes a result for the event. The event loop can then apply to the respective callback to obtain the desired instruction for the application-level behaviour. The pattern constrains these three sets of components’ purpose and communication ability to other elements. The event queue (and event sources) are unaware of the state of any thread in the worker pool and rely on the event loop to behave as an intermediary between them. Additionally, worker threads are constrained by the event loop directly. They can only process an event directly dispatched to them by the loop and perform no work otherwise.

**The Consequences:** Node.js benefits from the modification to the Reactor design pattern, seen in Twisted. The use of worker threads increases performance in processing events. Worker threads work synchronously, and therefore, they only focus on one task at a time, and many of these threads can parallelize the amount of work that Node performs. This modification contrasts Twisted's approach, which would rely on the developer to create separate event handlers in multiple threads to achieve the same result. Node's automatic spawning of these worker threads dramatically improves its efficiency over the Twisted implementation. Compared to Twisted, Node.js does suffer from a dependency on the JavaScript language. While JS is efficient for web applications, it may not provide Twisted's flexibility, which largely relies on Python and further extends to other languages.

### Flask Design Pattern: Plugin-based Reactor Pattern

**The Problem:** First off, Flask is a microservice framework for Python. Thus, it does not natively follow the Reactor-based pattern, nor is it built around asynchronous event-driven programming that leverages an event loop. The developers at Flask primarily built the framework around the idea of being minimalist and providing developers with a flexible framework to achieve desired outcomes. Therefore, it is possible to recreate a reactor-based environment in Flask, even though it lacks this pattern. The problem here describes a scenario where a developer wants to create an event-driven environment similar to those implemented in Node.js and Twisted. Event sources and event handlers need to be decoupled and provided with the proper resources to complete their tasks. Additionally, the system shall not block and wait on handling a single event, such that the program can efficiently handle multiple events at once.

**The Solution:** To achieve the Reactor design pattern, libraries are available as plugins to Flask. One helpful plugin for this purpose is to use [Gevent](http://www.gevent.org/), a "coroutine-based Python networking library that uses greenlet to provide a high-level synchronous API on top of the libev/libuv event loop." The developers at Gevent built the tool around the idea of using an event loop to handle events that come from the operating system. The event loop uses a polling register to determine when an event arrives. Once the event arrives, the event loop then reacts to them by running greenlets, which are thread-like structures that perform the processing of an event. Essentially, the event loop handles the dispatching of events to the event handlers, which are greenlets in Gevent. With this support, developers can create a non-blocking, asynchronous environment that can comfortably process events as they come in.

**The Participants:** Flask Gevent involves event sources, an event loop, and event handlers. Event sources are typically the operating system or any application-specific operations that trigger events upon a user I/O input. The event loop runs continually in the Gevent environment, and the event handlers are the greenlets. Greenlets only receive events through the event loop and do not communicate with the operating system or application-level components.

**The Consequences:** Flask Gevent benefits Python developers because it is a simple yet robust solution to creating an asynchronous environment that can handle multiple events. However, efficiency is in question because Flask does not natively support the Reactor pattern, as a Gevent library is required to achieve the desired behaviour.
