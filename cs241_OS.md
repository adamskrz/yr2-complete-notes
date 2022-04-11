# Intro to OS

What is an operating system – no universal definition, but generally software acting as intermediary between user and hardware.

The main OS responsibility ius to allocate resources to use processes and control their execution. E.g. control access to memory when it is used by multiple processes.

![OS responsibilities](OS_responsibilities.png?raw=true "OS responsibilities")

Services of an operating system can be split to 2 categories: useful to user (run programs, io operations, file system management) and useful to the system (resource allocation, security)  

**Program execution stages:** load program into memory, execute program, stop program.

**I/O operations:** write something to file, take input from keyboard/mouse/peripherals. User processes don’t control devices directly - instead, the OS controls them and allows multiple processes to access the same IO devices.  

**File system management:** allows programs to read/write files and other complex actions, subject to permission management.  

**Communications between processes:** OS allows such communications between processes on the same computer or through a network. The main ways to do this are sharing memory or direct messages.

**Error handling:** caused by hardware failure, IO devices or bad software. The OS handles the error to prevent a full system crash and generally stops the process that created the error. Sometimes the OS cannot prevent a full system crash if the error occurs in a critical process.

**Resource allocation:** scheduling CPU time and other hardware resources to different processes to allow multiple processes to run concurrently. Considers type of process, speed of hardware, number of jobs to be executed etc.

**Accounting:** keeps track of which processes are running and the amount of resources used, useful for system admin for billing purposes or to find misbehaving processes.

**Security:** concurrent processes cannot interfere with each other or the OS itself (purposefully or otherwise), and must keep the system secure - process's shouldn't be able to access other process's memory space unless authorised.

**Kernel**: the core of an OS, loaded into memory at startup and always running – provides necessary services like memory management, process scheduling, file handling.  

**Kernel space**: part of memory which stores kernel, kept separate from user space where user processes are. The split is to prevent user processes from interfering with the kernel for both security and stability. User processes can access kernel space through system calls that require permissions, giving limited access if required and the user process is trusted.

**System calls:** an interface where the user processes can request services (privileged operations/instructions) from the kernel. System calls provide access the low-level functions done only by the OS, e.g. hardware resource management, file operations, IO device access, etc, and can be limited to only authorised processes to increase security.

**Dual mode operation:** the hardware operates in kernel mode or user mode depending on if the operations are requested by the OS or user processes. Privileged instructions are only executable in kernel mode as they could be abused by malicious user processes. System calls change the mode to 0/kernel, and the return from a call changes it to 1/user.

![OS kernel mode](OS_kernel_mode.png?raw=true "OS Kernel Mode")

# OS structure

OS structure is large and complex.

**Early OS:** simple structures with not well defined levels, e.g. several layers could access IO devices. Hardware didn’t support dual mode operation.

![Early OS/Unix structure](OS_early_unix.png?raw=true "Early OS/Unix structure")

**Early UNIX:** huge amount of functionality built into the single kernel layer. Very little overhead in system calls, but hard to maintain and modify due to interdependencies. These monolithic structures have elements that have carried over into modern OS.

**Layered approach:** each layer interacts with the layers above and below, capped by the hardware and user processes. Layer K uses the services of layer K-1 and provides services to layer K+1. Modular design allows for easy layer modification, as the services they require and need to provide are already specified, so can easily modify the implementation without changing functionality. However, layer definition can be difficult – it must be ensured that layer K+1 doesn’t require services from layers below K-1. Also, more kernel layers mean more system calls between layers, increasing the overhead of services provided to the user.  

**Microkernels:** modular design of kernel, remove all non-essential services from kernel and implement them as user services, e.g.

![OS microkernel services](OS_micro_kernels.png?raw=true "OS microkernel services")

**Advantages**:

- Easy to extend the kernel due to simplicity
- Security and reliability are better as there is less to go wrong in the kernel  

**Disadvantages:**

- Performance suffers as there is more user-kernel processes interaction, so more system call overhead

**Loadable kernel:** the most modern approach and used in Windows/Linux/MacOS. A micro-like kernel implements the core services while other complex kernel services are loaded and run as is required. Similar to layered as each module is specifically designed for one purpose, but any module can interact with any other, so easier to access other kernel services.

# Processes

**What is a process:** a program currently in execution (in memory). One application can create several processes.

**Process in memory:**

- Text – stores instructions
- Data – stores the global and static variables
- Heap – dynamically allocated memory
- Stack – stores local variables and function parameters e.g. return address of the function

**Process states:**

- New – process is being created
- Running – instructions from the process are being executed
- Waiting – the process is waiting for some event to occur
- Ready – the process is waiting to be assigned to a processor
- Terminated – process has finished execution

State change graph:

![Process state change](Process_state_change.png?raw=true "Process state change")

**Process Control Block (PCB):**

- OS maintains a PCB for each process – data structure tracking the process
- Keeps track of:
  - Process state – running, waiting, ready, etc
  - Program counter – location of next instruction in process
  - CPU registers – contents of the CPU registers for use in case of interruptions
  - CPU scheduling information – priorities, scheduling queue pointers
  - Memory management info – locating process in the memory
  - Accounting info – CPU time used, time since start
  - IO status – list of files/IO devices in use by process

### Switching between processes

![Process switching diagram](Process_switching.png?raw=true "Process switching diagram")

**Process Scheduling:**

- To maximise CPU use, processes are switched from on/off as others wait
- Process scheduler selects among available processes for next execution on CPU
  - Job queue – set of all process in the new state (long term scheduler)
  - Ready queue – set of all processes in the ready state (short term scheduler)
  - Device queue – set of processes waiting for an IO device

**Queues:** contains a head and a tail pointing to the next process to be serviced and the last one to be, with each process also pointing to the next in the queue. Processes can be inserted in any point in the queue depending on the priority by starting at the head and inserting when the new process has greater priority than the next in the queue.

![Process queue diagram](Process_queue_diagram.png "Process queue diagram")

### Queuing diagram

![Process queue flowchart](Process_queue_flowchart.png "Process queue flowchart")

## **Schedulers**

There are 2 main types of schedulers: short term and long term.

**Short term scheduler:**

- Selects the next process to be executed from the ready queue
  - Not necessarily FIFO - can be priority queue or linked list based on scheduling algorithm
- Invoked very frequently – at least once every 100ms
- Must be very fast – if it takes 10ms to decide a CPU burst on a process of 100ms, then 9% of CPU time is wasted

**Long term scheduler:**

- selects processes in the new state to be brought into memory and ready queue
- Infrequent – can take minutes between creating one process and the next
- Controls number of processes in memory – when stable the arrival rate = completion rate
- Processes can be IO bound (more time doing IO than computation – short CPU burst) or CPU bound (more time doing computation – long CPU burst) - the long term scheduler tries to get a good mix of these processes

Time-sharing systems like Linux/Windows don’t have a long term scheduler and dump all processes on the short term scheduler. Long term schedulers are only used in computational workstations or similar where maximising CPU usage is needed and fast response to user input isn't required.

**CPU Scheduling Goals:**

- **Maximise CPU utilisation** - proportion of time the CPU is busy (not idle)
- **Maximise Throughput** - Number of process that complete their execution per time unit
- **Minimise turnaround time** - Amount of time to execute a particular process
- **Minimise waiting time** - Amount of time a process waits in the ready queue
- **Minimise response time** - Amount of time taken from request submitted to first response (very important)

### **When does a scheduler schedule?**

**Non-preemptive scheduling:** A process voluntarily stops and gives scheduler the CPU.

**Preemptive scheduling:** A process is forced to stop and give the scheduler the CPU. In this case, a process may have been added to the ready queue and could have higher priority than currently running process so this must be checked. This can cause race conditions if a process is '*preempted*' (forced to stop) while updating shared data (outside a critical region) that is accessed by another scheduled process. 

- A process switches from running to waiting (non-preemptive)
- A process switches from running to ready (preemptive)
- A process switches from waiting to ready (preemptive)
- A process terminates (non-preemptive)

### **Scheduling algorithms**

**First come first serve (FCFS):**

- Tasks assigned to the CPU in order of arrival.
- Can lead to un-optimal average waiting time, varies greatly depending on arrival sequence.
- Never does preemptive scheduling.

**Shortest job first (SJF):**

- Process assigned in order of increasing CPU burst time.
- Always gives the smallest average waiting time, but may cause a long CPU burst time process a large wait time.
- CPU burst time for a process cannot be known: the scheduler estimates the length of time of each process.
  - This can be done using 'Exponential Moving Average'
  - ![Exponential moving average diagram](exponential_moving_average.png)
- Can be preemptive or not: if preemptive, then CPU is stolen and runs the scheduler each time a new process joins the ready queue (preemptive gives better results assuming 0 overhead).

**Priority scheduling:**

- Each process is given a priority and assigned in order of priority.
- Can be preemptive or not, like SJF.
- Starvation is an issue - very low priority processes may never get scheduled if there is a large amount of incoming processes.
  - '*Aging*' solves this my gradually increasing the priority of processes that haven't been scheduled.

**Round robin (RR):**

- CPU runs in small bursts of time quantum '*q*'. After this time, the process is preempted and added to the end of the ready queue.
- Generally new processes are added to the end of the queue - assigned in order of arrival.
- For N processes and time quantum *q*, each process gets 1/N of the CPU time in chunks of at most *q* time units at once. No process waits more than (N-1)\**q* after being added to the ready queue.
- If *q* is too large, then this becomes FCFS and if *q* is too small, then there are many context switches and high overhead.
- *q* is typically 10-100ms (where a context switch takes < 10us).

## **Process creation**

All processes are created by other processes, except the kernel process which begins at startup (named 'init'). The *'child processes'* can create more processes – this forms a process tree, the root of which is the kernel.

Processes are identified using a process identifier - the *'PID'* (an integer in Linux systems).

![Process tree in Linux](Process_linux_tree.png "Process tree in Linux")

**Resource (CPU, memory, file) sharing:**

- There are 3 options for resource sharing between parent and child processes
  - The parent and child can share all resources
  - The child can share a subset of the parents resources, e.g. CPU and not memory/file
  - The parent and child share no resources

**Execution options:**

- There are 2 options for how parents and child processes execute
  - Parent and child execute concurrently
  - Parents waits until children terminate

**Address space:**

- 2 options:
  - Child’s address space and program is duplicate of parents (text, data, heap and stack are copied, not shared)
  - Child loads a new program into address space (no similarities)

**fork():** C command to create child processes that is an exact copy of the parent (including data/heap/stack) and continues from the line after fork(). In the child process, fork() returns 0 and in the parent process fork() returns the PID of the child (always an integer above 0).

## **Process Termination**

Processes terminate automatically after their last instruction is complete. Processes can also be terminated using exit() (in C, or using return() in the main function), which returns a status value to parent – parent can catch this to determine the success of the child process. All resources held by a process are released after termination.

Between child process termination and parents receiving the exit code, the child is a *'zombie process'* - the resources have been released but the entry is still in the process table. Once the parent receives the exit status of the child, the entry is removed from the table and the child is fully terminated.

Parent processes can terminate the execution of child process using abort() (in C) call. This is used when child takes up too much resources, no longer required or similar.

### **What happens when a parent exits before their children exit?**

Children become *'orphan processes'*. In UNIX systems, the init process is assigned as the parent process. The init process periodically issues wait() (in C) to collect the exit status of zombie orphan children to release their PIDs and entries in process table. Some OS don't support orphan processes (or child process must be specified to continue even if the parent exits), in which case orphan processes are immediately executed and removed from the process table.

*'Cascading termination'* is when a parent process exits, causing their children to exit, causing their children's children to exit, etc.

## **Inter-process Communication**

Processes running concurrently may want to communicate with each other for various reasons.

- Sharing files
- Sharing data about the user activity in the process
- Etc

There are two methods:

### **Shared memory**

A region of memory shared by communicating processes is created. This shared memory is typically inside one of the processes' memory and other processes requests access to it. System calls are used to create the shared memory, but there is no further use of kernel - there is low overhead. However, significant effort by programmer is required to maintain synchronization – ensure processes don’t try to access the same address at the same time.

![Shared memory between processes](Process_shared_memory.png "Shared process memory")
![Shared memory between processes](Process_shared_memory_2.png "Shared process memory")

### **Message passing**

Messages are directly exchanged between the processes using a communication channel provided by the kernel, e.g. a buffer. Processes use system calls to read/write to the buffer. This simplifies job of programmer (as the OS programmer already wrote the hard bit) but has a larger overhead due to many system calls.

The required system calls are a minimum of send() and receive() to allow message passing, whose functions are pretty obvious.

Message passing has multiple implementations/options – direct/indirect, how send/receive is synchronised, buffer size, etc.

- Direct:
  - Processes name each other explicitly when sending messages by using PID or similar
  - Links are established automatically and is associated with only one pair of communicating processes
- Indirect:
  - Messages sent through ‘mailbox’ using mailbox ID
  - Processes can only communicate if they share a mailbox
  - A link is established if processes share a common mailbox
  - Links may be between many processes and pairs of processes may have several communication links

Synchronisation of system messages

- Non-blocking - messages are sent and read whenever
- Blocking – synchronous messaging where messages are only sent when they are expected and only attempted to be received when messages are known to be sent
- Blocking send – sender pauses processing until the message is confirmed to have been received
- Blocked receive – receiver pauses processing until a message is available to receive

Buffering – A communication link is the buffer where messages are stored after being sent and before being read.

- Zero capacity – the queue has length 0, so sender waits until recipient receives the message
- Bounded capacity – the queue has finite length and once the end is reached, the sender must wait until the recipient reads a message from the buffer. This can take the form of a circular queue.
- Unbounded capacity – sender never waits after sending message

**Producer-consumer paradigm:** Shared buffer/memory is filled by a producer and emptied by a consumer - one process only sends messages and the other process only receives them. Consumer operates on the data passed to it by the producer, so waits if there are no messages. Similarly, if the buffer space is bounded then the producer must wait until a message is removed before adding.

## **Ordinary pipes**

Pipes provide a simple mechanism for one-way communication through message passing. The producer sends their output to the input of the pipe, which outputs to the input of the consumer. The consumer is typically a child process of the producer.

In C, the pipe() command creates a pipe by creating 2 temporary files and linking them:

![Pipe example in C using pip()](Pipe_example.png "Pipe usage in C using pip()")

X is set to 10 above

**Named pipes:** Exist outside of process run-time, in a directory as a file. These do not require a parent-child relationship that is typically needed in ordinary pipes, and once established many processes can use it for communication.

# Threads

A '*thread*' is a unit of CPU execution: it counts the number of chains of execution in a process. e.g. a single-threaded process has one chain of execution running each line sequentially.

Each thread shares the same code & heap and have individual stacks, thread IDs & register sets.
![Thread diagram](Thread_diagram.png "Thread diagram")

Threads are an alternative to having multiple processes when multiprocessing is required that needs to occur on the same code/data e.g. separate threads for reading in data and processing the data. Thread creation is much more efficient (less overhead) than process creation due to more shared resources, being 30x faster in Solaris OS, as well as thread switching being faster than process switching (context switching).

Web-servers are an example of a good use of multithreading, where many clients will attempt to concurrently access the server. Each client could be handled by their own thread, allowing actual concurrent access through parallel execution. Threads are used and not processes, as the procedure of handling a client request is the same, so the code can be shared.

### **Thread advantages:**

- **Economy** - faster and more efficient that process creation and switching
- **Scalability** - allows for large number of concurrent tasks
- **Responsiveness** - allows continued execution of other threads if there is a section waiting on IO/data input/etc
- **Resource Sharing** - threads share heap, so easy communication between threads

### **Concurrency VS Parallelism**

**Concurrency:** More than one task making progress - a single CPU can achieve this by interleaving process execution so they appear to be running at the same time.

**Parallelism:** Performing more than once task simultaneously (implies concurrency) - requires a multicore CPU.

**Data Parallelism** - distributes subsets of a data set across multiple cores, performing the same type of operation on each subset/core. e.g. summing contents of an array.

**Task parallelism:** - performing different tasks on the same data set simultaneously. e.g. finding the sum and mean of an array.

![Task and data parallelism](task_data_parallelism.png "Task and data parallelism")

### **Amdahl's Law**

![Amdahl's law](Amdahl_law.png "Amdahl's law")
Where P = proportion of process that can be parallelised using N cores and there is no overhead from adding parallelism.

## **Thread Synchronisation**

Threads must be synchronised in accesses to shared variables/memory/files. This is to ensure that changes made by threads are not overwritten by other threads with older versions of the data.

![Failed thread synchronisation](threading_issues.png)

This is a '*race condition*', where the last thread to finish writes their value to the shared variable and only that write matters. Proper thread synchronisation eliminates all race conditions.

### **Mutex locks**

Mutex locks synchronise threads by forcing them to acquire a lock before performing operations on shared variables. A mutex lock can only be acquired by one thread at a time, and so other threads wait at the point of acquiring the lock and continue once they have it.

![Mutex lock example](mutex_lock_example.png)

### **Semaphores**

### **Conditions**

## **Thread types and Threading models**

There are two different thread types: user threads and kernel threads.

### User-level threads

- Implemented by user programs in the user space
- Kernel is not aware of their existence and does not manage them, no kernel access overhead
- Cannot run in parallel on different CPU cores, but can run concurrently

### Kernel-level threads

- Implemented by the kernel in the kernel space
- Kernel can schedule them on different CPU cores - true parallelism
- Managed by kernel, so increased kernel overhead

### **Threading Models**

**Many-to-one:** Many user-level threads are mapped to a single kernel thread.

- Advantages
  - Only a single kernel thread, little overhead
- Disadvantages
  - Cannot run in parallel due to only a single kernel thread
  - A blocking thread causes all threads to stop until unblocked

**One-to-one:** Each user-level thread has its own kernel thread. (Used )

- Advantages
  - Other threads can run while one is blocking
  - Threads can run on different cores in parallel
- Disadvantages
  - If many kernel threads are created then the overhead becomes large and can slow the system

**Many-to-many:** Many-to-one, but there are multiple kernel threads with unique processes mapped to them.

- Advantages
  - The number of kernel threads and amount of user threads mapped to each can be varied in order to increase performance and reduce overhead (still more than many-to-one)
  - Kernel threads can run in parallel
- Disadvantages
  - User threads can still be blocked if running on the same kernel thread
  - Complex to implement

## **Server threading models**

- One thread per request
  - Each new request creates a new thread to handle it
  - High overhead as threads are repeatedly created and destroyed
  - High traffic will create large numbers of threads, slowing down the system
  - Can potentially handle any amount of clients concurrently - no queues
- Thread pool
  - Fixed number of 'worker' threads that handle requests from a queue
    - More efficient than creating and destroying threads
    - Number of threads can be optimised for a specific system to ensure no system slow down from too many threads
  - Requests are added to the queue by the main server thread
  - Synchronisation is required to ensure that no threads attempt to handle the same request and that a request isn't left unhandled

## **Signal Handling**

Used to notify a process about occurrence of events. A signal is generated by a particular event, the signal is delivered to the process it applies to and then the process handles the signal.

**Synchronous:** Internally generated by the process, e.g. runtime logic errors like division by 0 or illegal memory access.

**Asynchronous:** Externally generated, e.g. terminating the process from the command line.

Every signal has a default handler in the process provided by the kernel, e.g. the default handler for SIGINT terminates the process. '*user-defined*' signal handlers can be created by the process that overrides the default handler. Any signal handlers defined by the process can only use '*signal safe*' commands.

In multi-threaded programs, signals can be delivered to all or specific threads. *Synchronous* signals are generally sent to only the thread generating the signal, while *Asynchronous* signals are usually sent to all threads, e.g. SIGINT.
