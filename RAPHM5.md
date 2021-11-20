# Milestone 5: Reverse Engineering Programming Styles

Name: Raphael Bassot
Course: SENG350 - Software Architecture
Group: #2

# Part 2: Additional System Programming Styles - Node.js
### Programming Style #1 - Cookbook Style

Twisted implements the Cookbook programming style in their code, ["where the larger problem is divided into *procedures* each doing one thing"](https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C004.xhtml#c04_sec001). The codebase demonstrates examples of utilizing shared global variables to save state and have this state be accessed by different subroutines in the same module.

Similar to Twisted programming style, Node.js also employs the Cookbook style in specific areas in the codebase. The concrete problem situation that Node.js solves with this style is creating a basic blocking program for an SSL connection to be served, in a file named `saccept.c`. This file can be found in the codebase at the following relative path: `/deps/openssl/openssl/demos/bio/saccept.c`.

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