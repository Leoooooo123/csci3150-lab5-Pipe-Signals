#`kill()` system call

Don't think that `kill()` is to terminate a process only. It can send all kinds of signals.


```c
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
```

The following code demonstrates the use of `kill()`. Check the manual to see what `SIGSEGV` is?

```c
/* Signals/kill.c */
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <signal.h>
#include <unistd.h>


int main(int argc,char *argv[])
{
    printf("My PID: %d\n",getpid());
    sleep(5);
    kill(getpid(),SIGSEGV);
    return 0;
}
```