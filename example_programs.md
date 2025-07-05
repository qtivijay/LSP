## what is the output of the program

```

#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <sys/wait.h>

int main() {
    int x =20;
    pid_t pid = vfork();  // Child shares address space with parent
    if (pid == 0) {
        // This replaces the process image with /bin/ls
        x = 30;
        execl("/bin/ls", "ls", "-l", NULL);
        // If execl fails
        _exit(1);

    } else {
      int status;
      pid_t terminated = waitpid(pid, &status, 0);  // Block until pid2 exits
      printf("x = %d",x);
    if (WIFEXITED(status)) {
        printf("child execution is done");
    }

```
