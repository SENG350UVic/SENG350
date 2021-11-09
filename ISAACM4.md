# Twisted: The Strategy Pattern

In Twisted, different reactors are choosed based on the operating system in order to leverage maximum efficiency when the reactor utilizes system hardware, filesystem structure, and other key device elements to schedule and perform tasks. Twisted uses the **Strategy** Design Pattern to accomplish this. This entails receiving run-time instructions as to which algorithm to use in a family of algorithms.

Twisted achieves this in two ways:
- A default reactor is chosen based on the operating system. Consider the following code excerpt from `twisted/src/internet/default.py`[[1]](#1):
```
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
	
<a id="1">[1]</a>
https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/default.py
