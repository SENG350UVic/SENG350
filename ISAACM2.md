# Isaac's Part Biatch

## Platform Abstraction

The Twisted event-driven engine abstracts the system platform (e.g. Windows, Mac) from the program and provides a consistent set of methods that perform different actions on different devices that would lead to the same effective change. This is accomplished by ['The most suitable default reactor for the current platform.'](https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/default.py)
The state that this abstraction contains is the reactor itself. Event-driven engines use reactors to simulate a repeated loop that performs scheduled tasks (called events).

One such example of a Twisted reactor is the [`epollreactor`](https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/epollreactor.py) which is designed for linux platforms. The operations for this abstraction can be seen in the linked code file as python methods. The most important operations for this abstraction concern descriptors for events in the event loop, identifying which data is readable and writable at any given moment to prevent multiple events from manipulating the same data when it is not safe to, and management of events. Since different operating systems have different internal software structure, Twisted's `_getInstallFunction` function provides a reactor that is optimized to both communicate with the platform's kernel and provide an interface consistent with the other reactors to the rest of the Twisted program. This abstraction stores information regarding reactors that are likely to work well for various platforms. Developers using Twisted can still choose a different reactor, meaning that this abstraction is not necessary. Potential performance issues can leak information regarding the inner workings of said reactors, but the purpose of this abstraction is to solve that problem for developers before it ever occurs.

This abstraction is used in many plugins within the Twisted framework, notably the [`reactors`](https://github.com/twisted/twisted/blob/trunk/src/twisted/application/reactors.py) plugin-based system which checks for available reactors and installs the best one based on the heurstics from the [`default`](https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/default.py) abstraction. By providing developers with a default reactor, developers need not worry about optimizing their code for their platform or performance hits when switching platforms / deploying to servers running on different platforms.

### Node.js

Node.js is not an event-driven engine. Rather, Node.js is a JavaScript runtime. However, it has an [`os`](https://github.com/nodejs/node/blob/master/src/node_os.cc) module which gathers information regarding the current operating system and assigns variables and functions with which the rest of the program can interact with the operating system. For example, it provides the number of CPUs on the device with the `getCPUs` method. This abstraction is the closest thing to Twisted's default reactor abstraction within Node.js's codebase.

The two abstractions are similar in that they abstract out operating system intricacies from the rest of the program. The only significant difference is that Twisted provides multiple reactors which can accomplish this task, whereas Node.js only has one `os` module. This means that developers using Node.js must rely on the only built-in option for operating system communication, whereas developers using Twisted have an array of reactors to choose from. Twisted clearly has the better abstraction for this reason, though it obviously required more work from the Twisted developer to enable such flexiblity.

### Flask

Flask appears to use Python's [`os`](https://docs.python.org/3/library/os.html) module, which was designed by the Python team. As a result, Flask does contain any extra software infrastructure beyond what is already available in Python to handle operating system communication. For example, in [`cli.py`](https://github.com/pallets/flask/blob/main/src/flask/cli.py), the `os` module is used to format string paths as they would need to be formatted for the operating system. Python separates thread-handling to the [`_thread`](https://docs.python.org/3/library/threading.html) module, which as of Python version 3.7, is now available by default in all Python applications.

Flask is designed to be 'simple but extensible'[1], which helps to explain this decision. Developers using Flask can use Flask with Python's `os` and `_thread` modules out-of-the-box, or they can add their own functionality via Flask extensions[2].

[1]: https://flask.palletsprojects.com/en/2.0.x/
[2]: https://flask.palletsprojects.com/en/2.0.x/extensions/
