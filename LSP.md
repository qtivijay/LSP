# Linus System 
### Linux system into 5 subsystems
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

    
