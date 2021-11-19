# Milestone 5: Reverse Engineering Programming Styles

Name: Raphael Bassot
Course: SENG350 - Software Architecture
Group: #2

# Part 2: Additional System Programming Styles - Node.js
### Programming Style #1 - Cookbook Style

Twisted implements the Cookbook programming style in their code, ["where the larger problem is divided into *procedures* each doing one thing"](https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C004.xhtml#c04_sec001). The codebase demonstrates examples of utilizing shared global variables to save state and have this state be accessed by different subroutines in the same module.

Similar to Twisted programming style, Node.js also employs the Cookbook style in specific areas in the codebase. The concrete problem situation that Node.js solves with this style is creating a basic blocking program for an SSL connection to be served, in a file named `saccept.c`. This file can be found in the codebase at the following relative path: `/deps/openssl/openssl/demos/bio/saccept.c`.

A portion of the source code that demonstrates the Cookbook style used in `saccept.c` can be seen below. `saccept.c` can be seen as the server-side handling of an SSL connection that is created between a client and server entity. The program uses a global `done` variable, that acts as a server interrupt flag to act upon interruption or duplicate calls to the `saccept` function. Should it be interrupted, the global flag is set to `1`, and the loop in the main function - that is continually accepting data from the client - breaks and terminates gracefully.

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

    while (!done) {
        ...
```


This small Node.js program benefits from the Cookbook programming style by allowing a resource to be shared, and by more clearly communicating in the main function what the program is achieving. ...