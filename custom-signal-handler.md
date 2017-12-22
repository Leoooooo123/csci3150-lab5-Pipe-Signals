# Custom signal handler

You can override the default signal handler to IGN (ignore) or a custom handler. This is useful if you would like your program to react for certain kind of signals.

The program below sets the signal handler of `SIGINT` to `SIG_IGN`. The OS will ignore the signal so it has no response to `CTRL-C`.

```c
/* Signals/sigign.c */
#include <stdio.h>
#include <signal.h>
int main(int argc,char *argv[])
{
    signal(SIGINT,SIG_IGN);
    printf("Put into while 1 loop..\n");
    while(1) { }
    printf("OK!\n");
    return 0;
}
```

---

To define a custom signal handler, you can create a function and set it with `signal()`.

```c
sighandler_t signal(int signum, sighandler_t handler);
```
```c
/* Signals/custom.c */
#include <stdio.h>
#include <signal.h>

void handler(int signal)
{
    printf("Signal %d Received.Kill me if you can\n",signal);
}

int main(int argc,char *argv[])
{
    signal(SIGINT,handler);
    printf("Put into while 1 loop..\n");
    while(1) { }
    printf("OK!\n");
    return 0;
}
```