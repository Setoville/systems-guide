# Systems Guide

## LINUX Commands

**ls**: List files in current directory. -l to view metadata and names. -i to view inodes and names.

**pwd**: Process working directory

**cat**: Catenate. merge and output to stdout

**pipe**: stdout of LHS into stdin of RHS

**more**: Filter for paging

**less**: Load chunks of a file at a time. Searchable

**chmod**: Change Mode. Change access permission of file system objects. rwxrwxrwx: user, group, other.

**echo**: Print parameter to stdout

**ftp**: File transfer protocol. Transfer files to/from a remote site

**head**: Show first N lines. Visually pipe into head

**tail**: Show last N lines. -f to slow-update

**grep**: Global Regex Print. Find regex in stdin

**wc**: word count

**sudo**: Superuser Do.

## Process commands

**top**: View system resource usage

**ps**: Runing processes in system

**kill 123**: Kill process with PID=123. 
- Sends **SIGTERM** signal to the process.
- kill -9 sends **SIGKILL**

**killall kubectl**: Kills all processes with name kubectl

**nice**: Amount that a process will yield. Lower nice = higher priority. -20 < nice < 20

# System Calls

## Process System Calls

**fork()** creates a duplicate of the currently running process. The return value of **fork()** is 0 for the child, and non-zero for the parent.

- Expensive to copy all data of a process
- The **memory table** is copied rather than the actual memory. Entries in the new memory table are marked **copy-on-write**. 

**exec()** loads a new program.
- Replace/discard address space, re-load with a new executable. 
- **if (fork() == 0), exec ("games/pong")** is a common practice

**clone()** creates a child process.
- Fork is for processes, clone is for threads.
- **fork()** just calls clone.

**wait(123)** blocks the parent process until the child process with PID=123 terminates (return value)

**exit(1)** terminates the current process with exit code 1.

**getpid()** gets current PID.

**getppid()** get parent PID.

## Signals System Calls (from a process)

**raise()** sends a signal to the calling thread.

**kill()** sends signal to a process. **killpg()** sends signal to a process group.

**sigprocmask()** examines and changes blocked signals.

**pause()** waits for a signal.

**pthread_kill()** sends a signal to a POSIX thread in the same process as the caller.

**semctl()** semaphore control. 

**sigreturn()** return from signal handler and cleanup stack frame.
- When kernel creates the stack frame for a signal handler, sigreturn() is placed into the stack frame. 
- Call undoes everything that was done.
  - Restore signal mask
  - Restore process' context (registers, processor flags)

## File System Calls

### File access
**open()** opens and possibly creates a file (or device). Returns a file descriptor.

**lseek()** move read/write file offset.

**mkdir()** creates a directory.

**rmdir()** deletes a directory.

**getdents()** gets directory entries.

**readdir()** reads directory entry.

**chown()** changes the owner of a file.

**chroot()** changes root directory of the calling process.

**pipe()** creates a pipe.

**link()** hard link. makes a new name for a file.

**symlink()** symbolic link. makes a new name for a file.

**stat()**

### File Descriptor Operations

**close()** close a file descriptor. 

**mmap()** maps files or devices into memory.

**dup()** duplicates a file descriptor.

**read()** reads from a file descriptor.
- readv for vector.
**write()** writes to a file descriptor.
- writev for vector.

**poll()** wait for some event on a file descriptor.

**mount()** mounts filesystem.

**brk()** changes data segment side.
- Change location of the program break; defines the end of the process' data segment.

# Network System Calls

**socket()**
**connect()**
**bind()** binds a name to a socket. Assigns the specified socket address

interrupts

# Processes

- All processes are created by another process
## Types of Processes
- Foreground process
  - Foreground processes are initialized through terminal sessions. 
  - There must be a user connected to start this process.
- Background process
  - Not connected to a terminal
  - Do not expect user input
  - Strart a process in background using &
  - **Daemons** 
    - Start at system startup and run forever.
    - Can be controlled by a user through INIT process
- init process
  - Mother parent of all processes on the system
  - Manages all other processes
  - Started by the kernel itself
  - Always PID=1
  - Adopts all orphaned processes
## States of a process
- **Running**
  - Process is ether running or ready to run
- **Waiting**
  - Process waits for an event to occur
  - Process waits for a system resource
  - Interruptable waiting processes may be interrupted
  - Uninterruptable waiting processes may not be interrupted
    - Waiting on hardware conditions.
    - Cannot be interrupted by events/signals.
- **Stopped**
  - Process is stopped
  - Usually from a signal
- **Zombie**
  - Process is dead
  - Entry still exists in the process table
## Process creation
1. Allocate a slot in the process table
1. Assign a unique PID to the child process
1. Copy of the process image to the parent. No shared memory
1. Increments counters for any files owned by the parent.
1. New process is in Ready To Run State
1. Return value of 0 goes to child process
## Process control block
- Central process management
- Datastructure containing everything the OS needs to know about a program.
- Usually will have:
  - Identifier. Unique ID associated with the process.
  - State.
  - Priority.
  - Program Counter*
    - Sore the address of the next instruction to be executed.
  - Register Data*
    - Store current values of the registers. 
    - Also called context data.
  - Memory Pointers. 
    - Pointers to the code
    - Data associated with process
    - Memory that OS allocated by request
  - I/O Status Information. 
    - Outstanding requests, files or I/O devices currently assigned to process
  - Accounting information
    - Process use of resources.
  
## System Calls in Processes
- The primary thing that defines an OS is what system calls are made available.
- System calls are  functions in OS codethat programs can invoke with a special instruction.
- Kernel code is placed in the TOP of the address space of every running program. (above the stack)
- Process itself cannot access these pages. They can only access the pages through a system call
- Uses the stack frame of the process.
  - Allows system calls to execute in the same context as the process
  - No need to context switch; don't have to swap out memory tables of current process
  - Able to be interrupted!


## Context Switching
- Program counter and Register data in the PCB are updated when a system call or process switch occurs
- Upon an interrupt or system call:
  - Save the state of the process into the PCB.
  - Re-load state of the new process from its PCB.

Process table
- Process signal mask

## Inter Process Communication

# Hard Drive and Memory
# Files
Directory entries
hard vs soft links

## Device Files


# Signals (Software Interrupts)

- **6. SIGABRT**: abort, abnormal termination.
- **8. SIGFPE**: floating point exception.
- **4. SIGILL**: illegal, invalid instruction.
- **11. SIGSEGV**: Segmentation Violation, invalid memory access.
- **5. SIGTRAP**:
- **2. SIGINT**: Interrupt. 
  - Default behaviour is to terminate.
  - Orderly, graceful shutdown.
  - User-initiated happy termination.
- **3. SIGQUIT**: dump core.
  - Terminate process and dump core.
  - User-initiated unhappy termination.
  - Allows user to abort process.
- **15. SIGTERM**: terminate request sent to program.
  - Allow process to clean up.
- **23. SIGSTOP**: pause signal.   
  - **Cannot be caught or ignored**
  - Pauses a process.
  - Shell uses pause and **SIGCONT** to implement job control.
- **25. SIGCONT**: continue signal.
  - Resumes a process.
- **9. SIGKILL**: kill.
  - Kill the process immediatelly.
  - **Cannot be caught or ignored**
  - No cleanup

Signal handlers can be specified for all signals except **SIGKILL** and **SIGSTOP**.

Nobody really uses signal() anymore. Depricated.

# Terminals and Shells

Job control

# Networking

# Web servers

# Databases

# troubleshooting
