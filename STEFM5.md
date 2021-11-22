# Part 1: Reference System Programming Styles - Twisted

## Programming Style #1 - Cookbook Style

Twisted uses the [Cookbook][1] programming style in the development of this framework. A Cookbook features many complicated foods and recipes that would be impossible to reproduce on a surface level by just looking at the dish. However, the author divides this process into trivial integral steps. Alone, these steps do not form anything meaningful, but when a cook follows all the steps in order, only then, the dish is made. A developer can model their solution similarly by solving a complicated software-based issue instead of cooking an entree. For developing and reading purposes, the Cookbook layout is beneficial in showing the user the proper order to call functions for object instantiation and operation without explicitly including the instructions in a comment.

The developers at Twisted use the cookbook style in several different sections in the framework's source code, but a good example is in the **Service** class in the [service.py][2] module, shown in the code segment below.

```python
class Service:
    """
    Base class for services.
    Most services should inherit from this class. It handles the
    book-keeping responsibilities of starting and stopping, as well
    as not serializing this book-keeping information.
    """

    running = 0
    name = None
    parent = None

    def __getstate__(self):
        dict = self.__dict__.copy()
        if "running" in dict:
            del dict["running"]
        return dict

    def setName(self, name):
        if self.parent is not None:
            raise RuntimeError("cannot change name when parent exists")
        self.name = name

    def setServiceParent(self, parent):
        if self.parent is not None:
            self.disownServiceParent()
        parent = IServiceCollection(parent, parent)
        self.parent = parent
        self.parent.addService(self)

    def disownServiceParent(self):
        d = self.parent.removeService(self)
        self.parent = None
        return d

    def privilegedStartService(self):
        pass

    def startService(self):
        self.running = 1

    def stopService(self):
        self.running = 0
```

Here the problem is instantiating and operating the service abstraction. As service in twisted is verbatim, a

Here the problem is instantiating and operating the Service abstraction. A Service in twisted is verbatim in the software context. Fundamentally it represents anything that has binary states, depicting start and stops. In the above code segment, the solution is the exact layout of the methods in the class. On instantiation, the code initializes the Service with default state, name, and parent process. Initially, it is good practice to test the state of the current Object as, for the most part, a developer sees things from a black-box perspective. The `__getstate__(self)` method offers insight to the developer about the internal state of the Object and which the Cookbook style lists as the first instruction. Next, a developer needs to set the name of the Service for future identification and thus is the following listed method, `setname(self, name)`. Then, the developer must decide if this Service runs alone or if it is a child process of another Service. The `setServiceParent(self, parent)` method allows the programmer to initialize the Service parent and is the next instruction in the Cookbook. Corresponding disowns parent method `disownServiceParent(self)` is next in the style if the previous call needs reversing. With all the instance variables assigned, the user is ready to start the process and can do so with the `privilegedStartService(self)` and the `startService(self)` methods. When the user wants to stop the Service if it is exhausted, they can call, `stopService(self)`, which moves the Object to its final state and is thus the last instruction in the Cookbook style layout for this class.

The constraints in the Twisted program segment are the same constraints that come with the Cookbook programming style. The class methods do not call other methods or jump to another subsection of the code. Each function is a sub-task and, when called repetitively in the correct order of operations, furthers the problem's solution.

The consequences of this solution add benefits to the Twisted system as far as readability; however, it can result in the trade-off by increasing the size of the code. The Cookbook style employs keeping macro-operations segmented, but segmenting into functions requires more lines, such as a function header and return values. Using this programming style for a large project could result in massive programming files. However, being an open-source web framework, Twisted values readability at its core so that if a user needs to understand the underlying code, it is not a stretch to read it from the source. Twisted implements the Cookbook style skillfully and strategically so that the trade-off is not too extensive and achieves the goal of readability and modifiability.

## Programming Style #2 - Abstract Things

Twisted uses the [Abstract Things][3] programming style in the middleware of this framework. The Abstract Things style divides a problem into a collection of operations that are useful for the solution. Since applications constrain systems individually, there is no concrete solution for a given procedure. For example, suppose the Abstraction defines a search operation, and the system is stack memory-bound. This Abstraction offers the developer a framework for the solution and not the solution itself. Instead of defining a generic recursive searching algorithm that optimizes speed, it leaves the developer to choose a search operation that does not have a recursive solution, saving stack memory.

Twisted is an event-driven framework for network application; thus, it has many generic system entities, including event reactors, handlers, and generators. However, what makes Twisted unique to other event-driven frameworks, like [Node.js][4], is its implementation. Having abstract solutions and definitions for setting up event-driven processes helps keep the developer on track, solving individual sub-problems systematically and, in their way, unique to their application. The source code has a massive file with all the user-level defined abstractions located in the [interface.py][5] module. Below is a subsection of this module.

```python
class IAddress(Interface):
    """
    An address, e.g. a TCP C{(host, port)}.
    Default implementations are in L{twisted.internet.address}.
    """


### Reactor Interfaces


class IConnector(Interface):
    """
    Object used to interface between connections and protocols.
    Each L{IConnector} manages one connection.
    """

    def stopConnecting() -> None:
        """
        Stop attempting to connect.
        """

    def disconnect() -> None:
        """
        Disconnect regardless of the connection state.
        If we are connected, disconnect, if we are trying to connect,
        stop trying.
        """

    def connect() -> None:
        """
        Try to connect to remote address.
        """

    def getDestination() -> IAddress:
        """
        Return destination this will try to connect to.
        @return: An object which provides L{IAddress}.
        """


class IResolverSimple(Interface):
    def getHostByName(name: str, timeout: Sequence[int]) -> "Deferred[str]":
        """
        Resolve the domain name C{name} into an IP address.
        @param name: DNS name to resolve.
        @param timeout: Number of seconds after which to reissue the query.
            When the last timeout expires, the query is considered failed.
        @return: The callback of the Deferred that is returned will be
            passed a string that represents the IP address of the
            specified name, or the errback will be called if the
            lookup times out.  If multiple types of address records
            are associated with the name, A6 records will be returned
            in preference to AAAA records, which will be returned in
            preference to A records.  If there are multiple records of
            the type to be returned, one will be selected at random.
        @raise twisted.internet.defer.TimeoutError: Raised
            (asynchronously) if the name cannot be resolved within the
            specified timeout period.
        """


class IHostResolution(Interface):
    """
    An L{IHostResolution} represents represents an in-progress recursive query
    for a DNS name.
    @since: Twisted 17.1.0
    """

    name = Attribute(
        """
        L{unicode}; the name of the host being resolved.
        """
    )

    def cancel() -> None:
        """
        Stop the hostname resolution in progress.
        """


class IResolutionReceiver(Interface):
    """
    An L{IResolutionReceiver} receives the results of a hostname resolution in
    progress, initiated by an L{IHostnameResolver}.
    @since: Twisted 17.1.0
    """

    def resolutionBegan(resolutionInProgress: IHostResolution) -> None:
        """
        A hostname resolution began.
        @param resolutionInProgress: an L{IHostResolution}.
        """

    def addressResolved(address: IAddress) -> None:
        """
        An internet address.  This is called when an address for the given name
        is discovered.  In the current implementation this practically means
        L{IPv4Address} or L{IPv6Address}, but implementations of this interface
        should be lenient to other types being passed to this interface as
        well, for future-proofing.
        @param address: An address object.
        """

    def resolutionComplete() -> None:
        """
        Resolution has completed; no further addresses will be relayed to
        L{IResolutionReceiver.addressResolved}.
        """
```

Interfaces are incredibly similar to abstractions and differ only in the number of implementations that a class can have of the two. Also, interfaces are constrained to only having abstract methods, but an abstract class can also have defined ones that a developer could overwrite later in the implementation. In other words, interfaces are purely conceptual entities.

The Abstract Things style allows the programmer to focus on solving and customizing the underlying works of the event-driven feature of the framework. By using the technique of Abstract Things, a programming team can divide the work between the software Architectures and the programmers. Initially, having a person design an abstract framework first can streamline the design process by saving the developer from getting distracted by the framework's macro problems, focusing on only the subproblems instead. Since the reactor design is incredibly modular, it helps define the operations first, which the above code segment features. For example, in the class `IConnector(Interface)`, the subsection of the included source code contains abstract methods described for solving the subproblem of connecting entities in an event-driven system. Here the developer only focuses on writing code for solving each approach of `stopConnecting()`, `disconnect()`, `connect()`, and `getDestination()` without the need of occupying their mind with the higher-level system.

The constraints in the Twisted code segment are the same as the Abstract Things programming style constraints. The code module defines over thirty different interfaces in which the developer chooses where to implement them. The developer implements them to an Object based on whether it fits the underlying functions of the said Object. There is little concern about how a specific operation works in an interface; instead, the importance is how well does the interface help define the problem. Only on implementation, the developer solves each operation's functionality. Here is where the programming style helps as each application's solution may differ, forcing the developer to define the functionality for each method per the problem's scope.

The implementation of Abstract Things programming style results in pros and cons for the Twisted framework. As mentioned before, using the technique allows a streamlined workflow from the architect to the developer. The Interfaces act as a divide and conquer kind of to-do list for the developers to segment their workflow. Usually, the divide and conquer strategy saves time in development and is therefore highly beneficial. However, the Abstract Things style increases coupling as many different sections of the source code implement abstractions. The higher coupling means that in the software design stage, architectures must make sure that they include everything necessary for the abstract entity before starting the development process. If a method is left out, this would require changes to propagate at a multimodule level, increasing turnaround time. Twisted implements this style well and defines their abstractions well so that this con does not outweigh the benefits of this programming style.

[1]:https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C004.xhtml "Cookbook Programming style - Chapter 4"
[2]: https://github.com/twisted/twisted/blob/trunk/src/twisted/application/service.py "Service Module Twisted"

[3]:https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C013.xhtml#c13_sec001 "Cookbook Programming style - Chapter 13"

[4]: https://github.com/nodejs/node "Node js github page"

[5]: https://github.com/twisted/twisted/blob/trunk/src/twisted/internet/interfaces.py "Twisted Interface Module"
