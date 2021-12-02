[25s]
Hi, my name is Isaac, [I'm Jay], [I'm Raph], [and I'm Stefan]. In this video, we'll be taking a look at three different software systems that provide cross-platform networking and web solutions for developers. For each of these systems, we'll analyze two significant design decisions and take a look at why they were made, as well as their implications on the system. The three software systems we'll be analyzing are Twisted, Node.js and Flask. 

[52s]
Our reference system is Twisted. Twisted is an event-driven networking engine built in Python. An event-driven engine involves a reactor that receives a stream of events from an event queue. Events, such as a device receiving a datagram or an object colliding with another object in a videogame, are binded to functions that perform relevant tasks, such as the system saving the contents of a datagram, or a character taking damage in a videogame. Event-driven programming differs from conventional imperative programming in that the main reactor loop is not in control of what happens in the program. Events decide what happens, and the main reactor simply calls functions to respond to the occurence of events. In this sense, an analogy can be drawn between the reactor, and the task scheduler on your device. Some popular applications that use Twisted are Twitch, The Apple Calendar, and the Ubuntu One file-hosting service.

[36s]
Node.js is a JavaScript runtime environment that runs on Chrome's V8 engine and executes JavaScript code outside of a web browser. It can be used for both front-end and back-end services. Similar to Twisted, node uses an event-driven model that allows functions to handle events. Where these two systems differ, is that they use different programming languages, and provide different levels of flexibility among their utilities. At the end of the day, both services can accomplish the same tasks with some tweaking. Some popular applications that use Node.js are PayPal, Netflix, and LinkedIn.

[30s]
Flask, unlike Twisted and Node.js, is a microframework for web development, and is written in Python. Flask differs from the other two systems in that it is designed to be lightweight and minimal, meaning it can be used from the get-go with no additional dependencies. Of course, like all good microframeworks, it is highly expandable with libraries and external tools for accomplishing a large variety of tasks that it does not natively support. Some popular applications that use Flask are Airbnb, Reddit, and Uber.