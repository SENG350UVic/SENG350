# SENG350 Stefan-M2: Reverse Engineering Architectural Decisions

## Twisted

Twisted uses the [IService Abstract][1] to represent any architectural entity that starts and stops. Typical items that fit this category include rendering scripts, network protocols, and real-time applications necessary for a web application to function. A developer that uses twisted can link predefined or custom services to their application cleanly and consistently, promoting usability and modifiability. Twisted allows multiple services to link to a single Application, as in most cases, robust web apps run several services to ensure a decent user experience.

Instances of this abstraction contain a name, running state flag, and their parent service. The name variable is set with the method, setName() and is the identifier of the service. With a unique identifier for each service, a Developer can store them in a data structure and then retrieve them by identification. This convenient and straightforward identifier allows program flexibility, making it easier for a wide range of employments. Then there is the running state flag, which is a boolean representing if the process is running or not. The startService() and stopService() methods set the running state flag and contain the instructions set by the developer to start the service itself. The IService abstraction also contains a startPrivilegedService(). This method contains the instructions for initializing a service and runs before the startService(). Lastly, the parent variable contains the entity from which the service starts, which the developer implements and sets by the setServiceParent() and disownServiceParent(). Having a parent-child structure of service calls helps process organization for easy maintainability.

In the source code, the Service abstraction plays a role in running Unix applications and Windows applications. Although the programs are not cross-OS compatible, Twisted supports both because of the IService abstraction's simplification. This abstraction frees the Software Architect's concern about program function and clarifies that linking guarantees compatibility as long as it starts and stops.

[1]: https://github.com/twisted/twisted/blob/trunk/src/twisted/application/service.py "service.py source code"


## Node

The closest abstraction to the IServce abstraction in Node.js is a [Process][2]. Since Node.js is an event-driven runtime, it handles [processes][3] concurrently and doesn't explicitly run them but instead listens for them. Until there is a request to the Node.js Application by said Process, Node.js remains idle or works on another process. However, a Process has methods to log, kill, change running directory, and much more functions than the IServerce. Although they both represent objects that run and perform a task, the Node developers design the Process as an object that works in a different model. The Process makes developing easier as it includes diagnostics for debugging and testing; however, it is limited by only having stop controls and cannot start without having a separate call to run it. Although this seems to limit the Developer, the creators at Node.js choose the essentialist approach, only including functions and features detrimental to the abstraction

[2]: https://github.com/nodejs/node/blob/master/lib/process.js "process implementation"
[3]: https://github.com/nodejs/node/blob/master/doc/api/process.md "process info and usage"

## Flask

The Flask abstraction is the closest to Twisted's IService. Flask uses the [Flask Abstraction][4] as the central object responsible for Application control, protocol management, and service management. Flask's broad range of functions makes it fundamentally different from IService, which in contrast, contains only the Process and Protocol control that run on the Application. Nowhere in IService do they have higher-level control of the Application. Unfortunately, Flask does not have a modular design, and therefore, the Developer must do the tedious work of looking up these functions to ensure they use them appropriately. However, this design allows for all the functional tools to exist in one place, so finding the source code does not require searching multiple directories. Having central control over your applications, with readily available tools, gives the Developer a user-friendly experience with the codebase.

[4]: https://github.com/pallets/flask/blob/main/src/flask/app.py "Flask Class"
