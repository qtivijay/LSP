# Linus System 
### Linux system divided into 5 subsystems
- File management subsystem
- Process management subsystem
- Driver/ Device management subsystem
- Memory management subsystem
- Network Management subsystem
#### Single program which controls all these subsystems is called *linux Kernel*
#### When linux kernel boots up , the first process that executes in the linux kernel is called init process

## Process Management subsystem
- what is process ..?
```
A Program running in user space of RAM is called process
```
- when you execute a file you will get executable (for example : a.out) , when a.out in execution is called process
- What are memory sections of a.out file ..?
  1. **.text**
    - Contains compiled **machine instructions** (code segment).
    - Usually marked **read-only** and executable.
  2. **.data**
    - Stores **initialized global/static variables**.
    - Loaded into memory as-is.
  3. **.bss**
    - Holds **uninitialized global/static variables**.
    - Occupies space only at runtime (zero-filled by OS).  
- When ever program gets into exection , the contents of a.out copies from Harddeisk to userspace of RAM.
- Total RAM is divided into two parts
  - Lower part of RAM is Kernel space of RAM
  - Upper part of RAM is user space of RAM.
- When a.out is executed , the contents of a.out (bss,data,text) are copied to userpsace of RAM.
- Two more segements gets added when a.out comes to execution i.e **heap** and **stack**.
  ```
   What are the contents of a program ..?
   Text , Data , BSS
  ```
  ```
  what are the contents of a process ..?
  Text , Data , BSS , Heap , Stack
  ```

**User Space Memory Layout of a C Process**

| Memory Region               | Description                                      | Growth Direction     |
|----------------------------|--------------------------------------------------|----------------------|
| **High Memory Addresses**  |                                                  |                      |
| Stack                      | Function calls, local variables, return addresses| ⬇️ Grows downward     |
| Memory-mapped files & shared libs | Loaded shared libraries, `mmap()` regions         | — (varies)           |
| Heap                       | Dynamic memory (`malloc`, `calloc`, etc.)        | ⬆️ Grows upward       |
| BSS (.bss)                 | Uninitialized global/static variables            | —                    |
| Data (.data)               | Initialized global/static variables              | —                    |
| Text (.text)               | Executable code (machine instructions)           | —                    |
| **Low Memory Addresses**   |                                                  |                      |

- when program comes to execution in **userspace** , at the same time for every process there will be block created in **kernel space** of RAM this is called as **Process control Block** (**PCB**) also called as **process descriptor block**. 
-  For every process **memory segements** are allocated in userspace of RAM and **process control block** is allocated in kernel space of RAM.
-  For every process there will be **unique** process control block in kernel space of RAM.
-  Each process will have its own **PCB**.
-  There will be common memory segments between processes but **PCB** is unique for each process.
- **PCB** is a structure and it contains information related to process.
  - PID (Process ID)
  - PPID (Parent process ID)
  - FD table (File descriptor table)
  - SDT (Signal disposition table)
  - Page table
- All these PCB's in kernel space are maintained as nodes in Double linked list.
- Process management subsystem is creating the PCB's and maintaining the PCB's in kernel space.
- **Scheduler** will have access to entire linked list in kernel space.
- **Scheduler** decides which program should run next. It depends on **Schudeling mechanism**
- There are two primary **Scheduling mechanism**
  - **round robin** scheduling - used by general purpose operating systems - windows, linux , android.
  - **preemtive priority based** scheduling - used for RTOS- FreeRTOS , VXworks.
-**round Robin** scheduling each process will get equal amount of CPU time for execution.
  - After expiration of execution of last process, scheduler jumps back to first process in round robin scheduling.
  **preemptive prioirty** based scheduling - if you have mulitple running processes unlike round robin these process will not get equal amount of CPU time ,the CPU time will be given to highest prioirty task. CPU time execute high priority task until it is done.
- Difference between round robin and preemptive based scheduling -   every new process is added at the end in round robin but in prioirty based scheuling any new task coming will execute based on priority value is called **nice value**. based on this nice value program will be place in one of the position in linked list.
- when high priority task comes to execution and takes over the CPU from lower prioty task is called **preemption**.
- **Advantage of round robin** - every process will get a equal amount of CPU time.
- **Advantage of preemptive priorty** - Highest priority task will be executed first with out waiting.
- **context switching**- switching one process to another process

### how to create a process programmatically ..?
- gcc file.c will give a.out
- by executing .\a.out will create a process

### how you will create another process from another process ..?
- **fork** system call
- if a process call fork then it create a new process called **child process**
### how does fork system call works ..?
- fork returns twice  - once in parent process and once in child process
- in parent process fork return child PID
- in child process fork return 0
- based on the return value we will decide which is parent process and which is child process.

```
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = fork(); // Create a child process

    if (pid > 0) {
        // This block is executed by the parent process
        printf("Parent: My child's PID is %d\n", pid);
    } else if (pid == 0) {
        // This block is executed by the child process
        printf("Child: I'm the child process, and my PID is %d\n", getpid());
    } else {
        // fork failed
        perror("fork");
        return 1;
    }

    // Both parent and child execute this
    printf("Common: PID %d continues execution here\n", getpid());
    return 0;
}
```
output:
```
Parent: My child's PID is 12345
Common: PID 12344 continues execution here
Child: I'm the child process, and my PID is 12345
Common: PID 12345 continues execution here
```
- fork() creates a duplicate of the current process.
- The parent gets the child’s PID as return value.
- The child gets 0, allowing you to distinguish between the two.
- Both processes continue from the same line after the fork() call, but take different paths based on the return value.
- when fork() system call fails it returns -1.
### what happens to memory segments and pcbs when child process got created ..?
- In old kernel (pre 2.2) , When fork() is called, the parent's memory segements is duplicated for the child in user space, and a new PCB is created in kernel space. (PCB is not same)
- In new kernel , when for() is called , new PCB will be created for child process in kernel space but memory segements in userspace initally will be shared by child process and parent process.
- when parent and child doing read operation they use same memory segement but not for write operation.
- If any process(parent or child) tried to write , first copy of memory is created and content which process wants to write will be copied in new memory location is called **write on copy technique**
- In the new kernel , utilization of the memory is more efficient.
### 🧠 fork() and Memory Behavior – Copy-On-Write (COW)

#### 🔹 Before `fork()`
+-------------------+
Parent Process
Text	Data	Heap	Stack
+-------------------+

#### 🔹 After `fork()` with COW
+-------------------+ +-------------------+ | Parent Process | | Child Process | |-------------------| |-------------------| | Shared Pages | <--> | Shared Pages | | Text | Data | Heap | Stack | +-------------------+ +-------------------+
(Note: Both processes point to the same physical memory pages.)

#### 🔹 On Write by Child (Copy-On-Write Triggered)

+-------------------+ +----------------------------+ | Parent Process | | Child Process | |-------------------| |----------------------------| | Original Pages | | Modified Copy of Page(s)  | Text | Data | Heap* | Stack | +-------------------+ +----------------------------+
(* Modified page copied and updated in child's page table)

---

### 🧪 Example

```c
int main() {
    int x = 10;
    pid_t pid = fork();

    if (pid == 0) {
        // Child process
        x = 20;  // Triggers copy-on-write
    } else {
        // Parent process
        printf("x = %d\n", x);  // Still prints 10
    }
}
```
output :
```
x = 10
```

### what happens in PCB when fork() is called

- Parent will have own PCB and child will have thier own PCB when a fork is called.
- They will not share the same PCB
- Most of contents of parent's PCB will be copied to child's PCB.
- not copied elements are - PID and PPID
  - example : child PPID will be parents PID and PID of child will be newly created
- FD table and SD table contents are copied to child PCB
- Signal disposition table will not be copied.
----
- Three file systems - which keep track of different components of kernel
  - /dev , /sys , /proc
- /proc will track of running process information.

#### how to know running processes information in the system .?
```
ps -ef
```
----
#### what happens when child process terminate ..?
- when child process terminate before parent , kernel informs parent about its termination.
- When child terminates it should return **exit code** to parent. purpose of exit code to know child process has terminated successfully or there in failure in execution.
- it is not mandatory to use exit system but it is good to use to let parent process know.
- using **exit(EXIT CODE)**
    - exit(0) - successful value
    - exit(non zero) - failure
 
### what happens when parent is terminated even before child ..?
- child will become **orphan process** if parent is terminated before child process.
- **init process** will become parent process of child process if parent process is terminated.
- Child process PPID will be updated with init process PID.
----
### If we want parent process to be terminated only after child process terminate, how to do it..? 
- **wait(&stat)** system call used for completion of child process.
- stat will have exit code of child process.
- wait is blocking call
- block will wait until child termiantes
- wait will suspend the execution of parent process until status information is recieved from the child.
- If you have many child process , parent process will not wait for all child process to complete , even if child process terminate using it stat code parent will also terminate. you have to use **wait** as many as you create child processes.
- **WEXITSTATUS** will help to extract exit status code.
- **wait** returns the pid of the child process which has been termianted
```
#include <stdio.h>
#include <stdlib.h>     // For exit()
#include <unistd.h>     // For fork()
#include <sys/wait.h>   // For wait()

int main() {
    pid_t pid = fork(); // Create child process

    if (pid < 0) {
        perror("fork failed");
        return 1;
    }

    if (pid == 0) {
        // Child process
        printf("Child process (PID: %d): Doing work...\n", getpid());
        sleep(2);  // Simulate some work
        printf("Child process exiting with status 42\n");
        exit(42);  // Exit with a custom status code
    } else {
        // Parent process
        int status;
        printf("Parent process (PID: %d): Waiting for child...\n", getpid());
        wait(&status);  // Wait for child to finish

        //WIFEXITED will return true if child exited normally with out any issue.
        if (WIFEXITED(status)) {
            int code = WEXITSTATUS(status);
            printf("Parent: Child exited with status %d\n", code);
        } else {
            printf("Parent: Child terminated abnormally\n");
        }
    }

    return 0;
}
```
output : 
```
Parent process (PID: 23016): Waiting for child...
Child process (PID: 23021): Doing work...
Child process exiting with status 42
Parent: Child exited with status 42
```
### Can wait system call wait for specific child process from multiple child process ..?
- To wait for a specific child process among several, you should use the **waitpid()** system call instead of the generic wait()
- wait(&status) waits for any one terminated child.
- When you've forked multiple child processes, wait() doesn't give you control over which one it waits for.
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid1 = fork();

    if (pid1 == 0) {
        // First child
        sleep(3);
        printf("Child 1 exiting\n");
        exit(1);
    }

    pid_t pid2 = fork();

    if (pid2 == 0) {
        // Second child
        sleep(1);
        printf("Child 2 exiting\n");
        exit(2);
    }

    // Parent waits for pid2 (second child) specifically
    int status;
    pid_t terminated = waitpid(pid2, &status, 0);  // Block until pid2 exits

    if (terminated == pid2 && WIFEXITED(status)) {
        printf("Parent: Child with PID %d exited with code %d\n",
               terminated, WEXITSTATUS(status));
    }

    // Then wait for the other child
    waitpid(pid1, &status, 0);

    return 0;
}
```
output : 
```
Child 2 exiting
Parent: Child with PID 23373 exited with code 2
Child 1 exiting
```
### 🧾 `waitpid()` Options Reference

| Option        | Description                                                                 |
|---------------|-----------------------------------------------------------------------------|
| `0`           | **Default blocking behavior** — Waits until the specified child terminates. |
| `WNOHANG`     | Returns immediately if the child has not exited yet (non-blocking wait).    |
| `WUNTRACED`   | Also returns if the child was stopped (e.g., via `SIGSTOP`).                |
| `WCONTINUED`  | Returns if a stopped child is resumed (e.g., via `SIGCONT`).                |

> Header: `#include <sys/wait.h>` is needed to use these macros.
---
- Process management creates PCB which has PID,PPID,Fdtable ,Pagetable , signal disposition table
- Individually these components of PCB is managed by subsystems , For example:
  - Fdtable will be managed by File menagement subsystem.
  - Pagetable will be managed by Memory Management subsystem.
### How pagetable is maintained ..?  or How memory management helps process to manage the memory ..?
- In Userspace when program loaded from harddisk to RAM , we have memory segments- text, data, bass , heap , stack.
- The complete memory segment of a process is divided into equal parts of **4kb** called as **virtual pages**.
- Physical RAM is also divided into equal sized parts of 4KB which is called **physical frames** or **page frames**.
- Virtual pages are refered using page numbers is called **virtual page numbers** , similary physical frames are also numbered.
- When the process is started these virtual pages are copied to physical frames.
- which virtual page is present in which physical frame ,this information is stored in **page table**.
- Virtual pages stored randomly in physical frames.
- when process is loaded in parts , some of the virtual pages will be loaded in physical frames and remaining virtual pages are stored in area of harddisk called **swap area**.
- In linux we called it as **swap area** and in windows it is called **backing store**.

### 🧠 Memory Layout and Page Table Mapping

#### 1️⃣ Virtual Address Space (User Space)

```
+----------------------------+
| Stack                     |  <-- high address
+----------------------------+
| Heap                      |
+----------------------------+
| .bss (uninitialized data) |
+----------------------------+
| .data (initialized data)  |
+----------------------------+
| .text (code segment)      |
+----------------------------+
| NULL                      |  <-- low address
```

#### 2️⃣ Page Table (Simplified View)

| Virtual Page Number | Physical Frame Number | Valid Bit | Protection |
|---------------------|------------------------|-----------|------------|
| 0                   | 5                      | 1         | R-X        |
| 1                   | 8                      | 1         | RW-        |
| 2                   | -                      | 0         | ---        |
| 3                   | 2                      | 1         | RW-        |
| 4                   | 1                      | 1         | RW-        |

💡 Here:
- Virtual page 0 is mapped to physical frame 5.
- Virtual page 2 is **not** present (perhaps swapped out).
- Protection flags represent Read/Write/Execute permissions.

#### 3️⃣ Physical Memory (4KB Frames)

```
+-----------------+  Frame 0
| Unused / Other  |
+-----------------+  Frame 1
| Stack Segment   |
+-----------------+  Frame 2
| Heap Segment    |
+-----------------+  Frame 3
| Kernel Space    |
+-----------------+  Frame 4
| Unused / Other  |
+-----------------+  Frame 5
| .text Segment   |
+-----------------+  Frame 6
| Kernel Stack    |
+-----------------+  Frame 7
| I/O Buffers     |
+-----------------+  Frame 8
| .data Segment   |
+-----------------+
```


### 🧩 How It All Works Together

- When a user-space process accesses an address, the **MMU (Memory Management Unit)** uses the **page table** to translate the **virtual page number** into a **physical frame number**.

- CPU always operates on virtual addresses.
- In page table we also information called **Permissions** which decides memory is readbale or writable.
    - For example - Txt segament parts in two physical frames , for running program the Txt segement ,  memory should not be written , so the permission of Txt segement memory is read only.
    - By default all the physical frames in RAM are read and writable , but when procoess is loaded in RAM like some virtaul pages loaded into physcial frames, will be applied with some permissions that information will be there in page table. 
-Pagetable contains permissions access information.
```
+---------------+-------------------+-------------------+
| Virtual Page #| Physical Frame #  |   Permissions     |
+---------------+-------------------+-------------------+
|      0        |         12        |       R--         | ← Text Segment
|      1        |         15        |       R--         | ← Text Segment
|      2        |         7         |       RW-         | ← Data Segment
|      3        |         4         |       RW-         | ← Heap (Growing up)
|      4        |         -         |       ---         | ← Not loaded (Swapped)
|      5        |         9         |       RW-         | ← Heap
|      6        |         2         |       RWX         | ← Stack (Growing down)
+---------------+-------------------+-------------------+
```
----
### Address Transalation
- When a variable is created it will have virtual address.
- when these virtual pages are loaded into physical frames , these virtual address are translated into physical address.
- when we do **&X** of a variable we will get the virtual address.
- Virtual address = pagenumber * size of page + offset
  - Page Number	Identifies which virtual page the address belongs to
  - Offset	Tells how far into that page the data is located

 ```
Step 1: Split Virtual Address into Page Number & Offset
--------------------------------------------------------

  Virtual Address (VA):       0x2A13  →  Binary: 0010101000010011
                             [Page#][ Offset ]
                             [  2A  ][  13  ]

  - Page Number (VPN):        0x2A = 42
  - Offset:                   0x013 = 19

--------------------------------------------------------

Step 2: Page Table Lookup
-------------------------
  Let's say Page Table Entry[42] → Physical Frame #5

--------------------------------------------------------

Step 3: Compute Physical Address
-------------------------------
  Physical Frame = 5 × 4KB = 5 × 4096 = 20480
  Offset = 19

  → Physical Address (PA) = 20480 + 19 = **20499 (decimal)** = **0x5013 (hex)**

```
----
- when new child process is created , pagetable of parent process PCB is copied to child process pagetable PCB.
- Same memory segments will be used by both parent and child process until there is write operation.

```  
| Time         | Parent Maps To     | Child Maps To      | Physically Shared? |
|--------------|--------------------|---------------------|---------------------|
| Before fork  | Frame#42           | —                   | No                  |
| After fork   | Frame#42 (COW)     | Frame#42 (COW)      | Yes                 |
| After COW    | Frame#42           | Frame#55            | No (copied)         |
```
- when child or parent process tries to write. COW will be applied - First new memory segement is created , page table is updated with new address and then contents will be copied.

----
### Exec family of calls
There are four exec family of calls
1. execl
2. execv
3. execlp
4. execvp
- These are system calls which used another version of fork called **vfork**.
- exec family of calls internally uses **vfork**.
- In **vfork** there will be no write-on-copy technique is applied when child process is created.
- exec family of calls uses vfork **to execute shell programs**.
- exec family of calls can replace the complete process image with a new program.
- The memory segments in userspace like data,text bss,stack ,heap are also called as **process image**.
```
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = vfork();  // Child shares address space with parent

    if (pid == 0) {
        // This replaces the process image with /bin/ls
        execl("/bin/ls", "ls", "-l", NULL);

        // If execl fails
        _exit(1);
    } else {
        // Parent waits until child calls exec or _exit
    }

    return 0;
}
```
### 📌 Syntax
```
int execl(const char *path, const char *arg0, ..., (char *)NULL);
```
- path: Full path to the executable (e.g., "/bin/ls").
- arg0: The program name as seen in argv[0] (e.g., "ls").
- arg1, arg2, ..., argn: Additional arguments passed to the new process.
- (char *)NULL: Mandatory terminator of the argument list.
