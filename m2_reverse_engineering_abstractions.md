# SENG350 Stefan-M2: Reverse Engineering Architectural Decisions

## Twisted

- What is the abstraction? What dose it represent, and what is its purpose?

Twisted uses the [Service Abstract][1] to represents any web service, program, or protocol that stops and starts. A developer can include predefined, or custom services, necessary to make it rn there Application to make it run. If there Application requirers network file transfer they can include the FTP as a service and link it to there application.

[1]: https://github.com/SENG350UVic/twisted/blob/trunk/src/twisted/application/service.py "service.py source code"

- A link to the top-level source file implementing the abstraction in the codebase
- What is the key state that the abstraction contains and the key operations it supports? Please describe the state and operations conceptually rather then simply listing the number of variables and methods and focus on only the key state and operations. What is the key information the abstraction stores? What operations can be done with this information?

Classes implementing this abstraction contain a name, running state, and parent. The name is set through methods setName() and is the identifier of the service. Then there is the running attribute which is a boolean of wether it is running or not. This is set by the two important state functions of startService() and stopService. It also contains a startPrivilegedService() this is the init for the service and runs before the startService(). Lastly is the parent, the parent is the upper service that calls another service and is set by setServiceParent() and disownServiceParent().

- What are some common scenarios in which the abstraction is used? In what ways dose the abstraction make the scenarios simpler?

The in the source code the Service abstraction plays a role in running Unix applications and Windows applications. This abstractions simplifies the extra entities that a service may have. If you can set up any service in this way you know that it will have a simple start and stop and that it will always work.

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
