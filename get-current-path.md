# Get Current Path

To get the current working directory (cwd), we can use a handy function: `getcwd()`.

```c
#include <unistd.h>
char *getcwd(char *buf, size_t size);
```

```c
/* Shell/getcwd.c */
#include <stdio.h>
#include <limits.h> // Needed by PATH_MAX
#include <unistd.h> // Needed by getcwd()
int main(int argc,char *argv[]){
    char cwd[PATH_MAX+1];
     if(getcwd(cwd,PATH_MAX+1) != NULL){
         printf("Current Working Dir: %s\n",cwd);     
     }    
    else{         
        printf("Error Occured!\n");
    }    
 return 0;
}

```