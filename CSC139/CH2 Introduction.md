## **What happens when a program runs?**

**Von Neumann model of computing**
It executes instructions. Many millions of times every second, the processor **fetches** an instruction from memory, **decodes** it, and **executes** it. After it is done with this instruction, the processor moves on to the next instruction and so on, and so on, until the program finally completes.

**Virtualization** - the OS takes a **physical** resource and transforms it into a more general, powerful, and easy-to-use **virtual** form of itself.
	Thus, we sometimes refer to the operating system as a **virtual machine**.

A typical OS exports a few hundred **system calls** that are available to applications. because the OS provides these calls to run programs, access memory and devices, and other related actions, we also sometimes say that the OS provides a **standard library** to applications.

Finally, because virtualization allows many programs to run (thus sharing the CPU), and many programs to concurrently access their own instructions and data (thus sharing memory), and many programs to access devices (thus sharing disks and so forth), the OS is sometimes known as
a **resource manager**. Each of the CPU, memory, and disk is a **resource** of the system; it is thus the operating system’s role to **manage** those resources, doing so efficiently or fairly or indeed with many other possible goals in mind.

## **Virtualizing the CPU**
The OS can put up an **illusion** that the system has a very large number of virtual CPUs. Turning a single CPU into a seemingly infinite number of CPUs and thus allowing many programs to seemingly run at once is what we call **virtualizing the CPU**.

A **policy** devices which program should run, given we have 2 programs we want to run. Hence the role of the OS as a **resource manager**.

## **Virtualizing Memory**
The model of **physical memory** is very simple. Memory is just an array of bytes; to **read** memory, one must specify and **address** to be able to access the data stored there; to **write** (or **update**) memory, one must also specify the data to be written to the given address.

![[Pasted image 20250510222918.png]]
We see from the example that each running program has allocated memory at the same address `(0x200000)`, and yet each seems to be updating the value at `0x200000` independently! It is as if each running program has its own private memory, instead of sharing the same physical memory with other running programs

Indeed, that is exactly what is happening here as the OS is **virtualizing memory**.  Each process accesses its own private **virtual address space** (sometimes just called its **address space**), which the OS somehow maps onto the physical memory of the machine. A memory reference within one running program does not affect the address space of other processes (or the OS itself); as far as the running program is concerned, it has physical memory all to itself. The reality, however, is that physical memory is a shared resource, managed by the operating system.

## **Concurrency**
We sometimes get strange results with multi-threading since not all instructions execute **atomically** (all at once).

## **Persistence**
In system memory, data can be easily lost, as devices such as DRAM store values in a **volatile** manner; when power goes away or the system crashes, any data in memory is lost. Thus, we need hardware and software to be able to store data **persistently**; such storage is thus critical to any system as users care a great deal about their data.

The hardware comes in the form of some kind of **input/output** or **I/O device**; in modern systems, a **hard drive** is a common repository for long lived information, although **solid-state drives (SSDs)** are making headway in this arena as well.

The software in the operating system that usually manages the disk is called the **file system**; it is thus responsible for storing any **files** the user creates in a reliable and efficient manner on the disks of the system.

In order to create a file, the program makes three calls into the operating system. The first, a call to `open()`, opens the file and creates it; the second, `write()`, writes some data to the file; the third, `close()`, simply closes the file thus indicating the program won’t be writing any more data to it. 

To handle the problems of system crashes during writes, most file systems incorporate some kind of intricate write protocol, such as **journaling** or **copy-on-write**, carefully ordering writes to disk to ensure that if a failure occurs during the write sequence, the system can recover to reasonable state afterwards.

## **Design Goals**
So now you have some idea of what an OS actually does: it takes physical **resources**, such as a CPU, memory, or disk, and **virtualizes** them. It handles tough and tricky issues related to **concurrency**. And it stores files **persistently**, thus making them safe over the long-term. Given that we want to build such a system, we want to have some goals in mind to help focus our design and implementation and make trade-offs as necessary; finding the right set of trade-offs is a key to building systems.

One of the most basic goals is to build up some **abstractions** in order to make the system convenient and easy to use.

One goal in designing and implementing an operating system is to provide high **performance**; another way to say this is our goal is to **minimize the overheads** of the OS.

Another goal will be to provide **protection** between applications, as well as between the OS and applications. Because we wish to allow many programs to run at the same time, we want to make sure that the malicious or accidental bad behavior of one does not harm others; we certainly don’t want an application to be able to harm the OS itself.

Protection is at the heart of one of the main principles underlying an operating system, which is that of **isolation**; isolating processes from one another is the key to protection and thus underlies much of what an OS must do.

The operating system must also run non-stop; when it fails, all applications running on the system fail as well. Because of this dependence, operating systems often strive to provide a high degree of **reliability**.

Other goals make sense: **energy-efficiency** is important in our increasingly green world; **security** (an extension of protection, really) against malicious applications is critical, especially in these highly-networked times; **mobility** is increasingly important as OSes are run on smaller and
smaller devices. 

## **Brief History**
1. **Early Days: No OS (1950s)**
- Computers ran one job at a time, manually loaded by operators.
- Programming was done in machine language or assembly, and the system was idle between jobs.

2. **Batch Systems (Late 1950s–1960s)**
- Operators started grouping jobs into batches to reduce idle time.
- Introduced simple OS features: automatic job sequencing, basic error handling, and spooling (managing I/O).

3. **Multiprogramming (1960s)**
- OSes really took off in the era of the **minicomputer**.
- Allowed multiple jobs in memory at once.
- CPU could switch between jobs when one was waiting for I/O, improving CPU utilization.
- Required memory protection and job scheduling features.

4. **Time-Sharing Systems (Late 1960s–1970s)**
- Introduced interactive computing: multiple users accessed the computer via terminals.
- The OS supported fast context switching, user isolation, and resource sharing.

5. **Personal Computing (1980s)**
- Rise of personal computers (PCs) made OSs like MS-DOS and later Windows popular.
- Simpler than time-sharing systems, focusing on a single user.

6. **Networking and Distributed Systems (1990s–2000s)**
- OSs started to support networking, enabling distributed computing and the Internet.
- New challenges: security, remote access, distributed file systems.

7. **Modern OS Features (2000s–Present)**
- The **personal computer**, or **PC**.
- Emphasis on parallelism, virtualization, cloud computing, and mobile devices.
- OSs must now handle multicore processors, virtual machines, and mobile constraints.

