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
 
  
