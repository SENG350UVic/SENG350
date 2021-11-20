# Milestone 5: Reverse Engineering Programming Styles

Name: Raphael Bassot
Course: SENG350 - Software Architecture
Group: #2

# Part 2: Additional System Programming Styles - Node.js
### Programming Style #1 - Cookbook Style

Twisted implements the Cookbook programming style in their code, ["where the larger problem is divided into *procedures* each doing one thing"](https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C004.xhtml#c04_sec001). The codebase demonstrates examples of utilizing shared global variables to save state and have this state be accessed by different subroutines in the same module.

Similar to Twisted programming style, Node.js also employs the **Cookbook** style in specific areas in the codebase. The concrete problem situation that Node.js solves with this style is creating a basic blocking program for an SSL connection to be served, in a file named `saccept.c`. This file can be found in the codebase at the following relative path: `/deps/openssl/openssl/demos/bio/saccept.c`.

A portion of the source code that demonstrates the Cookbook style used in `saccept.c` can be seen below. `saccept.c` can be seen as the server-side handling of an SSL connection that is created between a client and server entity. The program uses a global `done` variable, that acts as a server interrupt flag to act upon interruption or duplicate calls to the `saccept` function. Should it be interrupted, the global flag is set to `1`, and the loop in the main function - that is continually accepting data from the client - breaks and terminates gracefully. The difficulty with these tasks is having to handle them at the same time, while keeping the codebase legible and easy to follow along.

```
static volatile int done = 0;

void interrupt(int sig)
{
    done = 1;
}

void sigsetup(void)
{
    struct sigaction sa;

    /*
     * Catch at most once, and don't restart the accept system call.
     */
    sa.sa_flags = SA_RESETHAND;
    sa.sa_handler = interrupt;
    sigemptyset(&sa.sa_mask);
    sigaction(SIGINT, &sa, NULL);
}

int main(int argc, char *argv[])
{
    ...
    sigsetup();
    ...

    while (!done) {
        ...
```


To solve this problem, the Cookbook programming style is used. This minor Node.js program benefits from the style by allowing a global resource to be shared between subroutines, and clearly communicating what the program is achieving. The `done` global variable is declared at compile-time, and is continuously checked in the main function, and modifiable by subroutines `interrupt()`, reached by a call to `sigsetup()` during main function execution. Doing this allows Node.js to achieve a shared state of the program at a specific instance, where the main function can react appropriately to these changes in the state (the `done` variable). Additionally, Node.js simplifies the flow of control throughout the main function by hiding the details of the server-side setup and interrupt-handling parts of the code. The `sigsetup()` subroutine instantiates the required variables and holds them in a struct, to be used by the external library function - signal.h's `sigaction()` -  that detects the occurrence of an interrupt. This type of abstraction is called *procedural abstraction*, where the function has been abstracted away through a natural use of subroutines to partition the work. This Node.js program can primarily focus on the SSL connection being handled, as the interrupt-handling work has been hidden from a developer that reads through the main function. The abstraction shown here reduces the complexity that a new developer encounters while trying to understand the program for the first time, and additionally cleans up the program by having distinct subroutines to tackle the different goals of this program - namely creating and serving a new server-side SSL connection, and handling intterupts should they occur.

The constraints in this Node.js program are the same constraints that bound the Cookbook programming style. Some of them have been discussed previously: the requirement to use subroutines to control flow of execution, and global variables to share a common state between these subroutines. Additionally, the Node.js program is restricted to omit large jumps or GOTO statements in code, which is the source of unreadable spaghetti code. The Cookbook style is constrained for the right reasons; it allows developers to focus on the functionality of their program by providing the peace of mind that, when followed, the Cookbook style guarantees for readable and modifiable code.

The consequences of this solution that implements the Cookbook programming style is identical to those of Twisted middleware. Both Node.js and Twisted utilize the Cookbook style and therefore earn the same benefits. The Cookbook style allows both products to have understandable code and a concise main function that abstracts away inessential details through subroutines. Node and Twisted both take advantage of a clear flow of control, but are certainly constrained by the use of one or more global variables that are shared throughout the different methods of the program. Node.js and Twisted have concrete examples of the Cookbook implementation, where code is clear, well-partitioned into subroutines, and makes use of shared global variables.


### Programming Style #2 - Abstract Things

Twisted middleware makes use of the **Abstract Things** programming style in their source code. Similarly, Node.js employs the **Abstract Things** style in their code to decouple the abstract object and its concrete implementation, and create reusable code.

Node.js is built on top of the V8 JavaScript engine, whose main purpose is to ["translate JavaScript code into executable machine code"](https://blog.appsignal.com/2020/07/01/a-deep-dive-into-v8.html). Relying on C++ to perform computations, the V8 engine is very fast and allows Node to load web browser information quickly. On top of this, V8 also needs to handle webpage visuals and HTML, and therefore a large toolset was created. Among these tools is Turbolizer - an HTML-based code visualizer for the phases throughout the Turbofan optimization pipeline. Turbofan is an optimizing compiler for machine code, and Turbolizer allows developers to visualize the work performed by Turbofan in a formatted HTML webpage. The main problem that the Turbolizer needs to solve is how to handle multiple types of views that can be analyzed (and displayed in HTML) during the code compilation. Basic information, code chunks, and graphs need to be displayed to the developer; the various view types share many attributes, but are unique in a way.

The V8 engine solves this problem through the use of Abstract Things. The Turbolizer leverages a base `View` abstract class, that is implemented in multiple ways for the numerous views that the tool requires. The abstract class and all concrete implementations can be found in the following directory: `deps\v8\tools\turbolizer\src\`. One implementation - the `CodeView` - can be partially seen below, alongside the abstract class, both of which are written in TypeScript.

```
export abstract class View {
  protected container: HTMLElement;
  protected divNode: HTMLElement;
  protected abstract createViewElement(): HTMLElement;

  constructor(idOrContainer: string | HTMLElement) {
    ...
  }
}
...

export class CodeView extends View {
  broker: SelectionBroker;
  source: Source;
  sourceResolver: SourceResolver;
  codeMode: CodeMode;
  sourcePositionToHtmlElements: Map<string, Array<HTMLElement>>;
  showAdditionalInliningPosition: boolean;
  selectionHandler: SelectionHandler;
  selection: MySelection;

  createViewElement() {
    const sourceContainer = document.createElement("div");
    sourceContainer.classList.add("source-container");
    return sourceContainer;
  }

  constructor(parent: HTMLElement, broker: SelectionBroker, sourceResolver: SourceResolver, sourceFunction: Source, codeMode: CodeMode) {
    super(parent);
    ...
```

The Abstract Things style allows the V8 engine to easily solve the issue of instantiating multiple types of views. The base abstract class contains two attributes - `container`, `divNode` - that are inherited by the subclasses, as well as an abstract function, `createViewElement()`, that is overridden in the CodeView implementation. The View parent class defines the fields that are required for each type of view, regardless of which type. Understanding how HTML pages work can clarify the reason for these two attributes; 'div' elements are container elements, which can contain text/images and form the basis of HTML webpages. The concrete implementation goes further, and defines the needed attibutes for a `CodeView` - a `codemode`, `source`, and `sourceResolver` among others. This represents the use of the Abstract Things style, in that the problem of displaying multiple view types is broken down into a parent `View` abstraction that is instantiated in many ways, like `CodeView`. The Turbolizer tool becomes easier to understand, as each type of view component directly relates to a parent class, and has attributes and a function that are abstracted from the developer reading through the subclass code.

The V8 engine tool, Turbolizer, is constrained by the Abstract Things programming style. Each type of concrete view element - such as the `CodeView` above - has become bound to the abstract class directly through TypeScript's `extend` keyword which denotes inheritance. The `CodeView` class relies on inheriting two of its class attributes from its parent, the `View` class. Any change to the parent class will change the behaviour of each implemented view component in Turbolizer's source code. Turbolizer's subclasses can therefore be easily extended, but not easily reduced, as they are bounded by the abstraction of their parent class.

Turbolizer benefits from the Abstract Things style by creating decoupled code and *Is-A* relationships. Decoupling in this scenario occurs because the concrete class inherits the methods of its parent, but overrides them which allows for customization of that method definition and as a result less dependency on the superclass. Each child class represents a smaller portion of the complete problem domain for Turbolizer. The main goal is to provide well-formatted, HTML-based visual representation of a compiler execution to the developer, which has many moving parts. The abstraction and implementations allow Turbolizer to be separated into distinct pieces of code that each tackle a purpose or represent a component of the system. Furthermore, the hierarchy creates *Is-A* relationships from the concrete implementation to its abstract parent. `CodeView`, intuitively, is a type of `View`, and therefore inherits parent attributes. The overall relationship between involved classes becomes easier to visualize for developers.

Node.js and Twisted share the same pros and cons with respect to their use of the Abstract Things programming style. This style benefits from more decoupled code through method overrides, which reduces inter-component dependencies. The amount of code that is reused is increased, because one abstract class definition takes part in the declaration of all of its subclasses. This was seen in Turbolizer's `CodeView` implementation of the `View` abstraction. Lastly, both Node.js and Twisted's use of the Abstract Things style allows for more understandable code that is partitioned into smaller pieces - each of which represent different types, components, or purposes that the system has.
