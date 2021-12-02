# H1: M6: Presentation Script

Name: Jigyasa Chaudhary  
V00899304  
Group: 2

## **Part 2: Design Decisions in Twisted**

### **Producer/consumer problem of handling shared resources; becomes difficult with too many threads**

- Twisted is an event-driven networking engine written in Python. In event-driven programming paradigm, program flow is determined by external events; characterized by an event loop and the use of callbacks to trigger actions when events happen. Twisted implements the reactor design pattern, which describes demultiplexing and dispatching events from multiple sources to their handlers in a single-threaded environment.

- The core of the engine is a reactor event-loop which knows about network, file system, and timer events. It waits on and then handles these events, abstracting away platform-specific behavior and presenting interfaces to make responding to events in the network stack easy.

Add code:

```python
while True:
    timeout = time_until_next_timed_event()
    events = wait_for_events(timeout)
    events += timed_events_until(now())
    for event in events:
        event.process()
 ```

- The reactor indicates event completion using a callback. The callback describes how to handle an event once it has completed. The event loop polls for events and dispatches them as they arrive, to the callbacks that are waiting for them. This allows the program to make progress when it can without the use of additional threads.

- Twisted hence counters the producer/consumer problem because the programmer doesn't have to worry about thread safety here which additionally makes it great for applications that require mutable data.

### **Various libraries in diff languages, across multiple protocols, must be used to create client-server implementations (complexity & dependencies are issue here)**

- Twisted is an engine for producing scalable, cross-platform network servers and clients, designed to be very flexible, and it lets us write powerful clients, at the cost of a few additional layers in the way to writing said clients. Twisted comes with client and server implementations for all of its protocols, as well as utilities that make it easy to configure and deploy production-grade Twisted applications from the command line.

**Why not use pre-existing applications?**

- Those server implementations have networking code written from scratch, typically in C, with application code coupled directly to the networking layer. This makes them very difficult to use as libraries.

- The clients and servers in Twisted are written in Python using a consistent interface. This makes it is easy to write new clients and
servers.

- Additionally, the scratch server implementations have to be treated as black boxes when used together, giving a developer no chance to reuse code if they wanted to expose the same data over multiple protocols. Extending these applications and maintaining cross-platform client-server compatibility is hard. On the other hand, Twisted's implementation makes it easy to share application logic between protocols.

- The server and client implementations are often separate applications that don't share code in traditional implementations while Twisted allows for easy client-server code sharing.
