# Twisted: The Strategy Pattern

In Twisted, different reactors are choosed based on the operating system in order to leverage maximum efficiency when the reactor utilizes system hardware, filesystem structure, and other key device elements to schedule and perform tasks. Twisted uses the **Strategy** Design Pattern to accomplish this. This entails receiving run-time instructions as to which algorithm to use in a family of algorithms.[[2]](#2)

Twisted achieves this in two ways:
- A default reactor is chosen based on the operating system. Consider the following code excerpt from `twisted/src/twisted/internet/default.py`[[1]](#1):
	```python
	try:
		if platform.isLinux():
			try:
				from twisted.internet.epollreactor import install
			except ImportError:
				from twisted.internet.pollreactor import install
		elif platform.getType() == "posix" and not platform.isMacOSX():
			from twisted.internet.pollreactor import install
		else:
			from twisted.internet.selectreactor import install
	except ImportError:
		from twisted.internet.selectreactor import install
		return install
	```
In reality, the decision made by the program at runtime is not easily readable from the code excerpt. It turns out that the `epollreactor` is specified on Linux, the `pollreactor` is specified on other non-MacOS POSIX-compliant systems, and the `selectreactor` is specified everywhere else (including MacOS and Windows).

- If a specific reactor is desired by the developer using Twisted, they can install only that reactor. Twisted will then use the reactor and override the default **if the installed reactor is compatible with the operating system**.[[3]](#3) Consider the following code excerpt from the Twisted documentation, which shows how to specify the `selectreactor` as the default reactor in Python:
	```python
	from twisted.internet import selectreactor
	selectreactor.install()

	from twisted.internet import reactor
	```

The default reactor based on the operating system is defined in `default.py` in the `_getInstallFunction` function.[[1]](#1) The different reactors are located in their own files and classes. One such example is the `epollreactor`.[[4]](#4) Each of these reactors is bound by multiple interfaces: from `twisted/src/twisted/internet/interfaces.py`[[5]](#5), `twisted/src/twisted/internet/posixbase.py`.[[6]](#6) Thus, each reactor is bound to the same set of methods and each achieve the same functionality via different algorithms, adhering to the Strategy Design Pattern.

This design, aligning with the Strategy Design Pattern, benefits from structural consistency within the codebase. Developers can use the interfaces provided for reactors to realize what methods are implemented, and developers can switch reactors or even operating systems without having to re-write any Twisted code (in the case of the event-loop).

<a id="1">[1]</a>
https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/default.py

<a id="2">[2]</a>
https://springframework.guru/gang-of-four-design-patterns/strategy-pattern/

<a id="3">[3]</a>
https://twistedmatrix.com/documents/13.1.0/core/howto/choosing-reactor.html

<a id="4">[4]</a>
https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/epollreactor.py

<a id="5">[5]</a>
https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/interfaces.py

<a id="6">[6]</a>
https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/posixbase.py

# Additional System Design Patterns

## Node.js

Node.js also contains the usage of an event loop. However, all event management is handled by Node.js. The reactors are not exposed to the developer. However, the methods for the reactors, such as adding and removing event listeners, are exposed, just as they are in Twisted. This implementation can be well-described by the **Chain-of-Responsibility** Design Pattern. This problem also exists in Twisted, but Node.js has no problem closer to Twisted's default reactor workflow than a problem that Twisted also has.

In the case of Node.js, this is solved via the `events.js` module, which provides functions for developers to *add and remove listeners* to the hidden underlying reactor. Without creating or modifying the reactor, developers using Node.js can ensure that their tasks are scheduled in the reactor.[[7]](#7)[[8]](#8) The main drawback to this workflow is that developers have no control over the efficiency of the reactor; the implementation of the underlying reactor is inaccessible to developers. Developers also cannot specify a reactor of their liking. This turns out to be a negligible issue for a large majority of developers.

<a id="7">[7]</a>
https://nodejs.org/api/events.html

<a id="8">[8]</a>
https://github.com/nodejs/node/blob/master/lib/events.js

## Flask

Flask does not inherently use an event loop. It turns out that, since Flask is a micro web framework and Twisted is an event-driven networking engine, Twisted can be used with Flask. This helps to explain why Flask, being designed to work out-of-the-box with minimal features, does not have an event loop by default. However, Flask developers can use asynchronous event loop modules outside of Flask, such as the `asyncio` module from Python.[[9]](#9) As with every Flask element discussed in this project thus far, Flask has always adopted the same solution of "let the developer add it on their own, if they really want it". As a result, there are no Flask-specific classes or files used to solve this problem. Rather, Flask treats an event loop as an extra opportunity for customization rather than a base feature. This, of course, means extra work for developers who want to use an event loop, but mitigating that work would go against the microframework ideology.

<a id="9">[9]</a>
https://docs.python.org/3/library/asyncio.html
