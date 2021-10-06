# SENG350 Stefan-M2: Reverse Engineering Architectural Decisions

## Twisted

Twisted uses the [IService Abstract][1] to represent any architectural entity that starts and stops. Typical items that fit this category include rendering scripts, network protocols, and real-time applications necessary for a web application to function. A developer that uses twisted can link predefined or custom services to their application cleanly and consistently, promoting usability and modifiability. Twisted allows multiple services to link to a single Application, as in most cases, robust web apps run several services to ensure a decent user experience.

Instances of this abstraction contain a name, running state flag, and their parent service. The name variable is set with the method, setName() and is the identifier of the service. With a unique identifier for each service, a Developer can store them in a data structure and then retrieve them by identification. This convenient and straightforward identifier allows program flexibility, making it easier for a wide range of employments. Then there is the running state flag, which is a boolean representing if the process is running or not. The startService() and stopService() methods set the running state flag and contain the instructions set by the developer to start the service itself. The IService abstraction also contains a startPrivilegedService(). This method contains the instructions for initializing a service and runs before the startService(). Lastly, the parent variable contains the entity from which the service starts, which the developer implements and sets by the setServiceParent() and disownServiceParent(). Having a parent-child structure of service calls helps process organization for easy maintainability.

In the source code, the Service abstraction plays a role in running Unix applications and Windows applications. Although the programs are not cross-OS compatible, Twisted supports both because of the IService abstraction's simplification. This abstraction frees the Software Architect's concern about program function and clarifies that linking guarantees compatibility as long as it starts and stops.

[1]: https://github.com/SENG350UVic/twisted/blob/trunk/src/twisted/application/service.py "service.py source code"

## Node

For each of the 3 abstractions you found in part 1, you should write 1-2 paragraphs for EACH of the two additional systems answering the following questions:

- What is the closest abstraction or abstractions to the abstraction in your reference system?

- A link to the top-level source file(s) implementing the abstraction(s) in the codebase

- In what ways is the abstraction different and the same? For example, does it use completely different elements that represent state and operations differently? In what ways are the operations supported by the abstraction similar? In what ways are they different?

- What are the pros and cons for the abstraction used in this system? In what ways does this abstraction make typical scenarios easier? In what does this abstraction make typical scenarios harder?

## Flask

For each of the 3 abstractions you found in part 1, you should write 1-2 paragraphs for EACH of the two additional systems answering the following questions:

- What is the closest abstraction or abstractions to the abstraction in your reference system?

- A link to the top-level source file(s) implementing the abstraction(s) in the codebase

- In what ways is the abstraction different and the same? For example, does it use completely different elements that represent state and operations differently? In what ways are the operations supported by the abstraction similar? In what ways are they different?

- What are the pros and cons for the abstraction used in this system? In what ways does this abstraction make typical scenarios easier? In what does this abstraction make typical scenarios harder?
