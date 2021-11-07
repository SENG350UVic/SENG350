# Milestone 4: Reverse Engineering Design Patterns

Name: Raphael Bassot
Course: SENG350 - Software Architecture
Group: #2

# Part 1: Reference Design Patterns
### Twisted Design Pattern: The Reactor Pattern

To achieve their goal of creating an event-driven engine, Twisted takes advantage of the **Reactor** design pattern. This pattern falls under the *Concurrency patterns* type, and is therefore outside of the Gang of Four design patterns. The Reactor pattern decouples the event source with the event handlers, and additionally does not let them interact with eachother; there is no subscribing, communication, or updating state between consumer and producer in this model. Instead, a central service handler acts as the intermediary, and directly handles the dispatching of events from their sources to their event handlers.

**The Problem:** In Twisted middleware, the event-driven architecture depends on how the events that are triggered upon completion of a task can be handled within the system. Events are essentially 'consumables', that need to get from producer to consumer for the system to properly react to a change. Events need to be waited upon from the source, then, when the handler is available to service it, the event should be sent to the event handler to invoke the proper chain of callbacks - the functions that perform the desired work in response to the event trigger. The network between event sources and handlers is complex. Events can be sent concurrently from multiple event sources in an application, and must be serviced properly by the correct handler as soon as it is available. Additionally, should one event be in the process of being handled, the system shall not block on other events and pre-empt the running handler. The harmonization of these tasks is a significant problem that Twisted must deal with.

**The Solution:** Twisted solves this problem by leveraging the Reactor design pattern. This design pattern separates Twisted into three parts - event sources, the service handler (or reactor event loop), and the event handlers. The reactor is the means of controlled communication between the event sources (such as a user input click) and event handlers (such as loading a new page). Twisted takes advantage of the event loop in this pattern to handle the flooding of concurrent events coming from the sources. The reactor event loop receives from the source, and has knowledge on the state of the ["network, file system, and timer events"](http://aosabook.org/en/twisted.html) for the system. Using this, the reactor can dispatch the events to the correct handlers at the next moment where the handler can start its synchronous operation on the event.

Twisted implements this pattern in their `twisted.internet` package, where developers can handle events efficiently in their single-threaded environment. An example is shown below, where the Twisted Reactor could be used is in an asynchronous URL fetcher. The reactor starts an event loop and receives any generated events (from the getPage() function) to then handle and dispatch them to the respective handlers (like the processPage() function). This example is taken from: http://aosabook.org/en/twisted.html

```
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

# Part 2: Addtional System Design Patterns
### Node.js Design Pattern: ???

### Flask Design Pattern: ???

