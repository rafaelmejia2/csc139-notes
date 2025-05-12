We virtualize the CPU to give the illusion that many virtual CPUs exist when we only have one physical CPU. This is called **time sharing** of the CPU.

**Program counter** tells us which instruction of the program will execute next. A **stack pointer** and associated frame pointer are used to manage the stack for function parameters.

**Process API** -- a real process API has these functions

**Create:** an OS must include some method to create new processes. OS is invoked to create a new process to run the program you have indicated.
**Destroy**: Systems should also provide an interface to destroy processes forcefully. 
**Wait**: We might want to wait for a process to stop running; thus some kind of waiting interface is often provided.
**Miscellaneous Control**: Most operating systems provide some kind of method to suspend a process (stop it from running for a while) and then resume it.
**Status**: Interface to get status information like: how long a process has run for, or what state it is in.

## **How does the OS get a program up and running?**

OS must load its code and any static data into memory, into the address space of the process. Programs initially reside on the disk in an executable format; thus, the process of loading a program and static data into memory requires the OS to read those bytes from disk and place them in memory somewhere.

In early operating systems, the loading process is done **eagerly**, i.e., all at once before running the program; modern OSes perform the process **lazily**, i.e., by loading pieces of code or data only as they are needed during program execution.

Once code and static data are loaded into memory, there are a few other things the OS needs to do before running the process. Memory must be allocated for the program's **run-time stack** for local variables, function parameters, and return addresses. The OS will also likely initialize the stack with arguments; specifically, it will fill in the parameters to the main() function, i.e., `argc` and the `argv` array.

The OS may also allocate memory for the program's **heap**. The heap is used for explicitly requested dynamically-allocated data; programs request such space by calling `malloc()` and free it explicitly by calling `free()`. The heap is needed for data structures such as linked lists, hash tables, trees, and other interesting data structures.

The OS also does other initialization tasks, particularly as related to I/O. In Unix systems, each process has three open file descriptors, for standard input, output, and error; these descriptors let programs easily read input from the terminal and print output to the screen.

## **Process States**

**Running**: A process is running on a processor.
**Ready**: A process is ready to run but for some reason the OS has chosen not to run it at this given moment.
**Blocked**: A process has performed some kind of operation that makes it not ready to run until some other event takes place. 

To track the state of each process, for example, the OS likely will keep some kind of **process list** for all processes that are ready and some additional information to track which process is currently running. The **register context** will hold, for a stopped process, the contents of its registers.

Sometimes people refer to the individual structure that stores information about a process as a **Process Control Block (PCB)**, a fancy way of talking about a C structure that contains information about each process (also sometimes called a **process descriptor**).

## **Summary**

- The **process** is the major OS abstraction of a running program. At any point in time, the process can be described by its state: the contents of memory in its **address space**, the contents of CPU registers (including the **program counter** and **stack pointer**, among others), and information about I/O (such as open files which can be read or written).
- The **process API** consists of calls programs can make related to processes. Typically, this includes creation, destruction, and other useful calls.
- Processes exist in one of many different **process states**, including running, ready to run, and blocked. Different events (e.g., getting scheduled or descheduled, or waiting for an I/O to complete) transition a process from one of these states to the other.
- A **process list** contains information about all processes in the system. Each entry is found in what is sometimes called a **process control block (PCB)**, which is really just a structure that contains information about a specific process.

# CH5 Process API

- Each process has a name; in most systems, that name is a number known as a **process ID (PID)**.
- The `fork()` system call is used in UNIX systems to create a new process. The creator is called the parent; the newly created process is called the child. As sometimes occurs in real life, the child process is a nearly identical copy of the parent.
- The `wait()` system call allows a parent to wait for its child to complete execution.
- The `exec()` family of system calls allows a child to break free from its similarity to its parent and execute an entirely new program.
- A UNIX **shell** commonly uses `fork()`, `wait()`, and `exec()` to launch user commands; the separation of fork and exec enables features like **input/output redirection**, **pipes**, and other cool features, all without changing anything about the programs being run.
- Process control is available in the form of **signals**, which can cause jobs to stop, continue, or even terminate.
- Which processes can be controlled by a particular person is encapsulated in the notion of a user; the operating system allows multiple users onto the system, and ensures users can only control their own processes.
- A **superuser** can control all processes (and indeed do many other things); this role should be assumed infrequently and with caution for security reasons.