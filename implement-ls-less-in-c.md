#Implement ls | less in C

In section 2.2, we see how pipe works for communication between parent and child processes. A familiar example of this kind of communication can be seen in all operating system shells.

pipe_lsless.c implements shell command "ls | less" in C.

```c
/* pipe_lsless.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc,char* argv[])
{
    int pipefds[2];
    pid_t pid,pid1;
    int status;
    if(pipe(pipefds) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
    pid = fork();
    if(pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }
    if(pid == 0) {//child
        close(pipefds[1]);
        dup2(pipefds[0],STDIN_FILENO);
        close(pipefds[0]);
        execlp("less","less",NULL);
    }
    else { //parent
        pid1 = fork();
        if(pid1 == 0) { //child
            close(pipefds[0]);
            dup2(pipefds[1],STDOUT_FILENO);
            close(pipefds[1]);
            execlp("ls","ls",NULL);
        }
        close(pipefds[0]);
        close(pipefds[1]);
        waitpid(pid,&status,WUNTRACED);
        waitpid(pid1,&status,WUNTRACED);
    }
    return 0;


}


```

##Analysis:

The parent uses the pipe for ls output. That means it needs to change its standard output file descriptor to the writing end of the pipe. It does this via dup2()system call and then executesls. The child will use the pipe for less input. It changes its standard input file descriptor to the reading end of pipe by dup2(pfd[0], 0). Then child executesless and the result is send to standard output. Thus the output of ls flows into the pipe, and the input of less flows in from the pipe. This is how we connect the standard output of ls to the standard input of less.