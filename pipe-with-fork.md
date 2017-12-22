# Pipe with`fork()` {#pipe-with-fork}

`pipe_creation.c`shows you the basic operation on pipe. But it is not very useful for a single process to use a pipe to talk to itself. In typical use, a process creates a pipe just before it forks one or more child processes. The pipe is then used for communication either between the parent or child processes, or between two sibling processes**\(A2\)**.`pipe_withfork.c`and`pipe_withfork2.c`show you how pipe works between related process.

In`pipe_withfork.c`, parent write "CSCI3150" to the pipe, child read from the pipe 1 byte at a time until the pipe is empty.

```c
/* pipe_withfork.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
int main(int argc, char* argv[])
{
    int pipefds[2];
    pid_t pid;
    char buf[30];
    //create pipe
    if(pipe(pipefds) == -1){
        perror("pipe");
        exit(EXIT_FAILURE);
    }
      memset(buf,0,30);
      pid = fork();
    if (pid > 0) {
      printf(" PARENT write in pipe\n");
    //parent close the read end
          close(pipefds[0]);
    //parent write in the pipe write end                 
      write(pipefds[1], "CSCI3150", 9);
    //after finishing writing, parent close the write end
          close(pipefds[1]);
    //parent wait for child                
      wait(NULL);                         
    }else {
      //child close the write end  
      close(pipefds[1]);   //-----line *
      //child read from the pipe read end until the pipe is empty   
      while(read(pipefds[0], buf, 1)==1)   
        printf("CHILD read from pipe -- %s\n", buf);
      //after finishing reading, child close the read end
      close(pipefds[0]);
      printf("CHILD: EXITING!");
      exit(EXIT_SUCCESS);
    }
    return 0;
}
```

![](/assets/pipe2.png)

**Analysis:**  
As you can see, the pipe works correctly. After fork\(\), pipe file descriptors are shared between the parent and child, as show in the picture below.

![](/assets/pipe_fork.png)

To ensure pipe work properly, you should:**\*Always be sure to close the end of pipe you aren't concerned with\*.**That is, if the parent wants to receive data from the child, it should close pipefds\[1\], and the child should close pipefds\[0\]. When processes finish reading or writting, close related file descriptors. Otherwise, there will be undesired synchronization problems.

pipe_withfork2.c is almost the same with pipe_withfork.c except line \* is commented out. That means, child forget to close write end. What will happen?

```c
/* pipe_withfork2.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
int main(int argc, char* argv[])
{
    int pipefds[2];
      pid_t pid;
    char buf[30];
    //create pipe
    if(pipe(pipefds) == -1){
        perror("pipe");
        exit(EXIT_FAILURE);
    }
      memset(buf,0,30);
      pid = fork();
    if (pid > 0) {
      printf(" PARENT write in pipe\n");
    //parent close the read end
          close(pipefds[0]);
    //parent write in the pipe write end                 
      write(pipefds[1], "CSCI3150", 9);
    //after finishing writing, parent close the write end
          close(pipefds[1]);
    //parent wait for child                
      wait(NULL);                         
    }else {  
      //child read from the pipe read end until the pipe is empty   
      while(read(pipefds[0], buf, 1)==1)   
        printf("CHILD read from pipe -- %s\n", buf);
      //after finishing reading, child close the read end
      close(pipefds[0]);
      printf("CHILD: EXITING!");
      exit(EXIT_SUCCESS);
    }
    return 0;
}

```
![](/assets/pipe3.png)

## **Analysis:** {#analysis}

We can see that the child process doesn't exit. What's going on? Note that the child process still has pipefds\[1\] open. The system will assume that a write could occur while any process has the write end open, even if the only such process is the one that is currently trying to read from the pipe, and the system will not report EOF**\(A4\)**, therefore. Thus child is waiting to read from pipefds\[0\], and the operating system doesn't know that no process will be writing to pipdfds\[1\]. So**the child process blocks until data arrives to be read\(A3\)**

