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

The default reactor based on the operating system is defined in `default.py` in the `_getInstallFunction` function.[[1]](#1) The different reactors are located in their own files and classes. One such example is the `epollreactor`.[[4]](#4) Each of these reactors is bound by multiple interfaces: from `twisted/src/twisted/internet/interfaces.py`[[5]](#5), `twisted/src/twisted/internet/posixbase.py`.[[6]](#6)

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
