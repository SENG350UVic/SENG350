# M6 Script

As Isaac mentioned, Flask is a Micro-web framework. This description essentially allows them to cop out of solving all problems associated with Architecture and leaves it to the user to decide on implementation.

## Problem 1 - Producer/consumer problem of handling shared resources; becomes difficult with too many threads

When it comes to solving the Producer/Consumer problem, there is no built-in solution that explicitly solves it. Flask only has the essentials to get up and running quickly with developing a website. So, say the user wants to implement a solution to this problem by directly editing the sources code. Editing the source goes against the framework objective. Thus, Flask implements an extremely dense, non-modular, and low interoperable design to prevent people from changing the framework. As seen in the image, there is exceptionally high coupling.

Instead of letting users edit the framework, Flask offers extensive support to many non-native plugins. It is entirely possible to use Flask and add twisted or node.js as a plugin and therefore take their solution to handle shared resources problems.

## Problem 2 - Various libraries in diff languages, across multiple protocols, must be used to create client-server implementations

The next problem has a similar solution; if you need to implement more languages, simply find a library that supports different languages and protocols.

Although this seems like a very abstract framework that offers no solutions, that is how the design there application. Flask lets the user make significant architectural decisions. It makes sure to support their choices and not get in the way.
