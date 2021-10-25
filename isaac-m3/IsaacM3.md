## Twisted

Twisted can be well-described as an event-driven architecture.
https://en.wikipedia.org/wiki/Event-driven_architecture
The biggest strength of event-driven architectures lies in it's ability to work with many loosely coupled components.
https://web.archive.org/web/20161002141057/http://esocc2016.eu/wp-content/uploads/2016/04/Leymann-Keynote-ESOCC-2016.pdf
Different events need not know anything about each other, including when they were fired, what information they transferred, and whether or not they were deferred or completed. By providing multiple types of reactors, as well as the ability to create your own reactor, Twisted enables developers using the software to both completely abstract their workflow from the system's resources via a process manager, as well as take full control of how system resources are distributed. This style; however, does pose constraints as well. Events often cannot communicate with each other directly or call each other's functions, as deferred objects can only be called once.
http://aosabook.org/en/twisted.html
In many cases, communication between such events can only be achieved via scheduled instances in the reactor that runs the event loop. This helps Twisted maintain it's event-driven paradigm: it is not built to forward the main process of handling events to an external source.