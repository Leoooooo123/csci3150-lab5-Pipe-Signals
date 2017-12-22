#Change Environment Variables

If you wish to change the environment variables, you can provide a new set directly in `exec*()` members, or by using `setenv()`.

```c
#include <stdlib.h>
int setenv(const char *name, const char *value, int overwrite);

```
Remember to set the overwrite flag to 1 in order to apply new configuration to existing variable!

```c
/* Shell/setenv.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>

int main(int argc,char *argv[])
{ 
    char *command1[] = {"shutdown",NULL};
    printf("Running shutdown.. it is in /sbin :P \n\n");
    setenv("PATH","/bin:/usr/bin:.",1); 
    execvp(*command1,command1);

 if(errno == ENOENT) 
    printf("No Command found...\n\n"); 
 else
    printf("I dont know...\n"); 
    return 0;
}


```