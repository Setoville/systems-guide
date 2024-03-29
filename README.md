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

**grep**: Global Regex Print. Find regex in stdin.

**wc**: word count of stdin.

**sudo**: Superuser Do.

**diff**: finds the line diff of two files.

## Process commands

**top**: View system resource usage

**ps**: Runing processes in system

**kill 123**: Kill process with PID=123. 
- Sends **SIGTERM** signal to the process.
- kill -9 sends **SIGKILL**

**killall kubectl**: Kills all processes with name kubectl

**nice**: Amount that a process will yield. Lower nice = higher priority. -20 < nice < 20

# LINUX System Calls

## Process System Calls

**fork()** creates a duplicate of the currently running process. The return value of **fork()** is 0 for the child, and non-zero for the parent.

**exec()** Replace current program with a new program.
- Replace/discard entire address space, re-load with new sections.
  - New code section
  - No heaps
  - No stack
  - Environment and file descriptors are **maintained**
    - Copied to a temp space, then copied into the new space.
- **if (fork() == 0), exec ("games/pong")** is a common practice.

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

**stat()** provides status of a file. (inode information)

### File Descriptor Operations

**close()** close a file descriptor. 

**mmap()** maps files or devices into memory.
- Allocate memory in a process
- Adds some pages into the process address space
- Maps those pages to resolve to addresses in physical memory
- **address = mmap(5000)** 
  - Makes the OS find a contiguous chunk of 5000 bytes
    - Allocates **pages**, not individual bytes.
  - Returns location of the start of the chunk.
- The OS keeps track of all allocated memory!
- Allocation is a tricky problem! Solutions to alloc/dealloc and defragmentation are hard.

**munmap()**
- De-allocates memory pages from address space
- munmap(address)

**dup()** duplicates a file descriptor.

**read()** reads from a file descriptor.
- readv for vector.

**write()** writes to a file descriptor.
- writev for vector.

**poll()** wait for some event on a file descriptor.

**mount()** mounts filesystem. Logically attach a filesystem to the main filesystem.

**brk()** changes data segment side.
- Change location of the program break; defines the end of the process' data segment.

## Networking System Calls

**socket()**
- Socket is an endpoint for communication
- Returns a FD that refres to the endpoint

**connect()** initiates a connection on a socket

**listen()** listens for connections on a socket

**bind()** binds a name to a socket. Assigns the specified socket address

interrupts

# LINUX Processes

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
    - Start at system startup and run forever, in the background.
    - Can be controlled by a user through INIT process
    - To daemonize a process:
      - fork()
      - setsid()
      - close(0), close(1), close(2).
      - fork()
  - **Service**
    - Application taht runs in the background
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
  - Return value has not been **reaped**
  
## Process creation, fork() and exec()
1. Allocate a slot in the process table
2. Assign a unique PID to the child process
3. OS makes a copy of the process image of the parent.
4. Increments counters for any files owned by the parent.
5. New process is in Ready To Run State
6. Return value of 0 goes to child process
- **Copy-on-write**
  - It is expensive to copy all data of a process.
  - The **memory table** is copied rather than the actual content in memory.
  - Entries in the new memory table are marked **copy-on-write**.
  - As soon as a process writes to that page in memory, the page is copied and the memory table is updated.

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
  - Memory Pointers and address code
    - Data associated with process
    - Memory that OS allocated by request
  - I/O Status Information. 
    - Outstanding requests, files or I/O devices currently assigned to process
    - File descriptors
    - Current and root directory
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

## Environment (env variables)
- The point of this is to **pass down** from one process to the next.
- Stored in the process itself, somewhere on the heap.
- Address of this location is stored as a variable in data section.


## Context Switching and Process Swapping
- 2 Reasons to context-switch
  - Multitasking. Usually triggered by I/O.
  - Interrupt handling. 
- **Recall:** User to kernel mode switch does not require a context switch.
- Program counter and Register data in the PCB are updated when a system call or process switch occurs
- Upon an interrupt or system call:
  - Save the state of the process into the PCB.
  - Re-load state of the new process from its PCB.

## Processes and Signals
- For each process, the OS maintains two integers with bits corresponding to signal numbers.
- Pending signals and blocked signals.
- A child created by fork() has an empty pending signal list, initially.

## Threads
- When a process runs, it has 1 thread by default
  - 1 code pointer, 1 execution stack
- Multiple threads; each have their own execution stack and code pointer.
- Share the same address space
  - Heap space can be seen by all threads
- Threads live on a process.
- Thread Control Block
  - Describes an execution context, PCB describes an environment context.
  - Pointer to the PCB of the process that the thread lives on.
  - Also has program counter?


## Inter Process Communication
- Why?
  - Information sharing
  - Computation sub-tasks
- Process may be **independent** or **cooperating**
  - Independant process may not be affected by other processes
- 2 models of IPC
  - Shared memory
    - Region of memory is shared by cooperating processes
    - All process read/write data to the shared region
    - Standard process:
      - **void* shared_memory = mmap(128);**
      - **memcpy(shared_memory, message, sizeof(message))**
  - Message passing (POSIX message queues)
    - Kernel manages a space of memory, usually as a queue.
    - **mq_open()** in C
- **Semaphores**
  - A signal used to control multiple access to a resource
  - MUTEX is used for exclusive access to a resource
  - Semaphore (even binary) should be used for synchronization.
    - A _turnstile_ allows one person at a time, but cna be locked at any time.

## Process Table
- Stores information about all currently running processes.
- Relates PID to PCB.
- Each PCB maps file descriptors to file pointers, entries in the **file table**


# LINUX Users and Permissions and Namespacing
- Every process gets a User ID
- User accounts are associated with permissions
- Superuser/root user gets UID=0
  - System calls will never fail
- Each file and directory is owned by a single user
- By default, fork and exec will pass down the user
- Sample login flow:
  - PID=1,UID=0 (init)
  - fork, exec
  - PID=2,UID=0 (login)
  - fork, setuid, exec
  - PID=3,UID=1780 (shell)
- User groups
  - A user may belong to many user groups
  - Permissions may be assigned to each group
  - Each file and directory is owned by a single group, as well as a user
  - rwxrwxrwx

## Permissions
- File permissions
  - Read: can read bytes of a file.
  - Write: can modify bytes of a file.
  - Execute: Can exec a file. Without this, exec call would fail.
- Directory permissions
  - Read: Can get names of files.
  - Write: Can add/remove/rename files.
  - Execute: Can use in file paths. Any system call with this directory NOT AS THE LAST DIRECTORY will fail
  
# LINUX Files and LINUX filesystem

## File read, write and descriptors
- Each file has a file descriptor
- Two steps for reading and writing.
  - I/O for a hard drive is a bottleneck
  - Data is copied to a _write buffer_ or _read buffer_, in memory, controlled by OS
  - OS writes from  _buffer_ to its destination
- NO verification of written data (no acks!)
- Each file descriptor has a marker of where currently reading/writing.
- Read buffer
  - Common practice: read portions in the file that follows!
  - Look ahead, read more than is requested so subsequent calls read from buffer

## Superblock
  - Record of characteristics of a filesystem
    - size, block size, size and location of inode tables
  - A request to access any file requires access to the filesystem's superblock.
  - Crucial! Backups are made and linux also maintains a copy of its superblock in memory.
  - The basic linux filesystem type //TODO


## Directories 
- Another type of file, users cannot update directories directly.
- Sequence of directory entries. Not sorted
- Maps file name to file's inode
- Symbolic links vs Hard links
  - Deleting a symbolic link does not affect the file.

## File System
- **I/O control**. Deal with device drivers and interrupt handlers to transfer data.
  - "read block 1234"
  - Usually controlled by writing bit pattersn in the I/O controller device file.
- **Basic File System**. Deal with physical blocks on a disk.
  - Defined by numerical physical addresses
  - Drive 0, cylinder 12, track 7, sector 1.
  - Responsible for buffers and caches
- **File Organization Module**. Aware of files and logical and physical blocks.
  - Translates logical block addresses to physical block addresses
  - Keeps track of free space (unallocated blocks)
- **Logical File System**. Manages metadata
  - File system structure, directory structure
  - File data is maintained in a _File control block_. This is called an INODE.

## INODES
- Disk is divided into equally-sized blocks
- Inodes do NOT store the file name!!
- Files that are too long get broken into multiple INodes
- Fixed length
- Stores metadata
- 12 direct pointers to blocks of data
- Single indirect pointer points to a block, which contains a table to more addresses
- Double indirect pointer points to a block, which points to two blocks, which points to more pointers.
- Map a file to its INODE. /foo/file.txt
  - inode for / -> data for / -> inode for /foo -> data for /foo -> inode for file.txt



# LINUX Signals (Software Interrupts)

- **6. SIGABRT**: abort, abnormal termination.
- **8. SIGFPE**: floating point exception.
- **4. SIGILL**: illegal, invalid instruction.
- **11. SIGSEGV**: Segmentation Violation, invalid memory access.
- **5. SIGTRAP**: trap. sent to a process when an exception or trap occurs.
- **2. SIGINT**: Interrupt. 
  - CTRL+C on terminal
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

Process Signal Mask is the collection of signals that are currently blocked.

Nobody really uses signal() anymore. Depricated.

# LINUX Terminals and Shells

Job control
producer consumer problem?
# LINUX Networking
- Literally just another file.
- Sockets are similar to pipes.
- Attach a socket to your program, just like attaching a mailbox to your house.
- Bidirectional with stream sockets, unidirectional with datagram sockets
- /etc/hosts is an internal DNS lookup
- Sockete example:
  - SSH daemon opens a socket on the machine on port 22
  - When the user wants to connect to the server using SSH, the user's SSH client connects to that socket.
  - Packets may now be exchanged
what is CURL, wget?

**traceroute()** 
- Send ICMP packet with TTL = 1, 2, 3
- When packet reaches TTL = 0, router reports back to origin
- Allows traceroute() to trace a network call
socket is one end of a network connection.

**ping()**
- Tests reachability of hosts on an IP network
- Sends an ICMP packet, and waits for an ICMP reply.

**ifconfig()**
- Lists all network interfaces
  - All associated IP addresses
- lo is the local loopback (localhost!)

**tcpdump()**
- Command line packet sniffer
- _tcpdump -c 10_ to capture 10 packets.
- _tcpdump -c 10 -i eth0_ to capture 10 packets on eth0 interface.

**netstat()** (network statistics)
- View active connections and routes
- _netstat -i en0_ for interface en0
- _netstat -r_ routing tables
- _netstat -c_ continuous output for all connections (TCP/UDP)
- _netstat -a_ for active connections
- _netstat -at_ for active TCP connections. _-au_ for active UDP connections.
- _netstat -tulpn_ TCP, UDP, Listening, show PID and program name, N for numeric.

## LINUX Sockets (Level 4)
- **Client side**
  - Create a socket with _socket()_ system call.
  - Connect to the socket of the address of the server with _connect()_.
  - Send and receive data. _read()_ and _write()_ work.
- **Server side**
  - Create a socket with _socket()_ system call.
  - Bind socket to an address with _bind()_.
    - Address is a port number on the host machine.
  - Listen for connections with _listen()_.
  - Accept connection with _accept()_.
  - Send and receive data.

# General Networking
**NAT**: Network Address Translation
- Allows networks to be hierarchical
- Local networks have private IP addresses
- **Overload (PAT)**
  - Packet arrives at the border
  - Router replaces privateIP:Port with the publicIP:Port (same port)
  - Place entry in the NAT table
  - Packet returns, NAT table lookup, route to privateIP
- **Dynamic NAT**
  - Manually create a pool of public addresses.
  - Packet arrives, takes first available public address
  - Packet returns, NAT table lookup
  - Public address returns to the pool
- **Static NAT**
  - Manually enter NAT table.
**DNS**: Domain Name Server 
- DNS server holds records of what domains point to what IP addresses
- ISPs have resolver servers that try to resolve.
- If the resolver server does not resolve, it calls the ROOT server
- ROOT server directs resolver to a TLD, top level domain server (.com, .net)
- TLD server directs request to the **Authoritative name server**
- Name servers know everything about the domain. They know!
- Resolver stores the IP address in cache

## Link Layer
- Flow control: Pacing between adjacent sending/receing nodes
- Error detection and Correction
- Half-duplex and full-duplex
- In each host, a Network Interface Card (NIC)
### Error Detection
- Single bit parity to detect errors in single bits.
- 2D bit parity to detect and correct single bit errors.
  - Check odd parity up-down, even parity left-right
- Cyclic Redundancy Check
  - "Long division". Pick a G that both src and dst know
  - Divide message D with G to get R, send DR.
  - dst divides DR with G, and checks if it matches R.

### Multiple Access Protocols
- Channel partitioning, random access and "taking turns"
- TDMA: Time-division multiple access
- FDMA: Frequency-division multiple access
- Random access MAC protocol.
  - CSMA/CD. Carrier sense, multiple access, collision detection
  - CSMA/CA. Carrier sense, multiple access, collision avoidance.
- **CSMA**: Listen before transmit
  - If channel is idle, transmit, else don't transmit.
- **Ethernet CSMA/CD**:
  - NIC receives datagram from network layer, creates a frame
  - If NIC senses channel idle, begin trasmission
  - If, during transmission, NIC senses a transmission, abort and jam
  - NIC enters **binary backoff**
- **Ethernet CSMA/CA**:
  - If NIC senses channel busy
  - Start random backoff time.
  - While channel is busy, timer is frozen
  - Timer counts down when channel idle
  - Transmit when timer == 0
  - If no ACK, increase random bakcoff interval, repeat sense.

### Mac addresses and ARP
- 32-bit IP address, 48-bit MAC address.
- MAC addresses are FLAT.
- **ARP**: Address Resolution Protocol
  - Each node gets an ARP table
  - Relates IP addresses to MAC addresses (Medium Access Control)
  - If MAC_A wants to send a datagram to MAC_B:
    - Look in ARP table. If not:
    - MAC_A broadcasts ARP query with B's IP address.
    - B receives, replies to A.
    - ARP table in A saves the IP address of B. with expiry.
- **Switch**: Link-layer networking device
  - Each switch gets a switch table
  - Switches are transparent and self-learning
  - **Store-forward**:
    - When a frame is received:
      - Record incoming MAC address!
      - Look into switch table with MAC dst address.
      - Forward frame on interface indicated by table entry
      - If no entry in switch table, FLOOD.
        - Forward on ALL interfaces except arriving.
### WLAN (WiFi)
- **Basic Service Set**
  - Access Point, Router, Computer
  - CSMA/CA
- Access point is a layer-2 device that converts wifi to eth.
- Modes of Operation of WiFi MAC protocol
  - Distributed Coordination Function
    - Handshake mode: Seek permission from receiver before sending a frame
    - No handshake mode: just send it.
  - Point Coordination Function
    - AP is the controller. AP decides who transmits and when
- Problems with WLAN
  - **Hidden Terminal Problem**
    - A collision occurs, but a node cannot detect, because it occurs outside its range.
    - Solved using CSMA/CA
  - **Exposed Terminal Problem**
    - When a node is prevented from sending packets to other nodes because of co-channel interference with a neighboring transmitter

## Network Layer
- Transport segments from src to dst host
- Encapsulate segments into datagrams
- **Forwarding**: Move packets from routers input to routers output
  - Getting through an intersection
- **Routing**: Determine route taken by packets from src to dst
  - Planning trip from src to dst
- **Buffering** is required when datagrams arrive from fabric faster than transmission rate.
- **Routers!!**:
  - Control Plane is responsible for routing and management (software)
  - Data plane is for forwarding (hardware)
  - Routers also have an ARP table.
  - **Routing Table**:
    - Contains routing information received through routing protocols and configuration
    - May be more than 1 entry for a given prefix.
    - A map of every network that router knows how to get to.
    - **The Longest prefix matching will win.**
  - **Forwarding table!**:
    - Computed using routing tables.
    - 1 defined "best" path for a given prefix.
  - Allow traffic isolation 
  - Routers have multiple IP addresses. 1+ for each physical device.
  - Routing Interface Protocol (RIP), Open Shortest Path First (OSPF), Border Gateway Protocl (BGP)


**DHCP: Dynamic host configuration protocol**:
- Assigns unique IP addresses to hosts on a network.
1. DHCP-Discover. Broadcast to all hosts on the network.
2. DHCP-Offer. DHCP server, presumably on the network, responds with an IP.
3. DHCP-Request. Host sends request to lease the address from DHCP server. 
4. DHCP-ACK. DHCP sends address, subnet mask, default gateway, DNS server.
- DHCP Server holds a lease time for each IP. They will eventually expire
- Maps IP addresses to MAC addresses as well. 

TCP windowing

## Special IP addresses:
- 127.0.0.1 is _localhost_
- Privat internets, not publicly routable VIA global internet
  - 192.168.0.0/16
  - 10.0.0.0/8
  
## Special ports:
- 20 FTP
- 22 SSH
- 80 HTTP
- 443 HTTPS

wifi and ethernet protocols

# Storage and Memory

## Page Tables
- Maps virtual memory to physical addresses
- Pages in a program's memory are mapped to portions in RAM and sometimes in hard drive, all over the place.
- The CPU's Memory Management Unit stores a cache of recently used mappings.
  - Translation Lookaside Buffer (TLB)

## Direct Memory Access (DMA)
- Hardware unit that assists with transfer between memory and I/O devices.
- Increment addresses for successive words

## RAID types

## LINUX Device Files
- UNIX uses device files to access hardware
- /dev
- **block** devices communicate by sending entire blocks of data
  - Hard disks, USBs
  - Large buffers
- **Character** devices communicate by sending/receiving single characters
  - Keyboards
  - Small buffers

# Web servers

# Databases

## CAP Theorem
- **Consistency**: Every read is the most recent write, or error.
- **Availability**: Every read will get a response. No guarantee of correctness.
- **Partition Tolerance**: System works despite failed network.
- _C + P_ Waiting for a response from a partition node might not be available, or respond.
  - Do this if you need atomic read/writes
- _A + P_ Request will get a response from a partitioned node, but the data may not be correct.
  - Do this if you ned to respond, or correctness is less important than availability.

## Consistency Patterns
- **Weak Consistency**: After a write, reads may or may not see it.
  - Best effort.
  - VOIP. Video chat.
- **Eventual Consistency**: After a write, reads will eventually see it.
  - Asynchronous data replication
  - Email, DNS.
- **Strong Consistency**: After a write, all reads will see it.
  - Synchronous data replication
  - File systems, transactions, DBMS.

## Replication Patterns
- **Master-slave replication**
  - One master, source of truth, replicates to many slaves. 
  - Clients may read and write from master, but only read from slaves.
  - Master is often **promoted**
- **Master-master replication**
  - More than one master
  - Clients may read and write from any node
  - Increased latency for synchronization
  - Sometimes violates ACID.

## Database Distribution
- **Federation**
  - Functional Partitioning
  - Splits up databases by function. There is no central master to serializa writes.
- **Sharding**
  - Splits up data such that each database handles a unique subset of data
- **Denormaliation**
  - Improve **read** performance by adding additional redundant information
  - Trades write time for read time


## Normalization
  - Removing redundant data from a database by splitting the table in a well-defined manner to maintain data integrity.
  - First normal form (1NF): When all entities of the table contain unique or atomic values.
  - Second normal form (2NF): If it is in 1NF and all non-key attributes of the table are fully dependant on the primary key.
  - Third normal form (3NF): If it is in 2NF and every non-key attribute is not **transitively dependent** on the primary key.
  - BCNF (3.5NF): No overlapping candidate keys.

## Locking (The readers-writers problem)
- Exclusive lock. 1 actor currently using the system. Nobody else can read/write.
- Shared lock. Many actors may read. Only when all the shared locks are gone, may an exclusive lock be applied.

## Data Indepedence
  - **Physical Data Independence**: Modifies schema at physical level without affecting schema at the logical level.
    - Changing indexing strategy without needing to make changes without logical level.
  - **Logical Data Independence**: Modifies schema at logical (conceptual) level without affecting or causing changes at the view (application) level.
    - Adding new entities or relationships to this schema should not need to have to re-write existing applications.

## Functional Dependency
- A relation is in functional dependency when one attribute uniquely defines another attribute.

## SQL
- **ACID**
  - Atomicity
  - Consistency
  - Isolation
  - Durabilirt
- A primary key is a field in a table which uniquely identifies each row/record in a table.
- Primary keys must be unique.
- A **superkey** is a set of attributes upon which all attributes are functionally dependent. No two rows can have the same value of superkey attributes.
- A **candidate key** is a minimal superkey. Superkey without redundant information.
- A **foreign key** is a field in one table that uniquely identifies a row in another table.
- A **Stored procedure** is a function that contains a set of operations compiled together.
- When multiple fields are used as a primary, this is called a composite key.
- SQL is derived from the **relational model**: All data is represented in terms of tuples, and grouped into relations.

## NoSQL
- Favour availability and scalability over consistency
- **BASE**
  - Basically Available
  - Soft State
  - Eventual Consistency
- **Key-value store**
  - O(1) reads and writes
  - High performance for simple data models
  - Redis, memcached
  - Usually 1 level deep. Less flexible
- **Document store**
  - JSON, XML, Binary
  - Documents organized by collections, tags, metadata, directories
  - Documents may have different keys.
  - More flexiblity over KVS.
  - MongoDB, DynamoDB.
- **Graph database**
  - Each node is a record, each edge is a relationship
  - Optimized to represent complex relations.
  - Many foreign keys and many-to-many relationships.
- **Wide-column store**
  - Nested map.
  - CassandraDB.

## SQL vs NoSQL
- SQL for:
  - consistency, guarantees.
  - complex queries
  - vertical scaling
  - structured data (schema)
  - high transaction-based applications.
- NoSQL for:
  - availability and scalability.
  - Simple queries that are FAST.
  - horizontal scaling
  - unstructured data
  - Hierarchical data storage.

# UNIX terminals and shells
- Computer sends text characters to the **terminal character device file**.
- Interact with a shell process using a terminal.
- Process puts data in output buffer of the device file, text is sent to terminal by OS. **shared memory**.
- Process expect to inherit FD=0 is STDIN, FD=1 is STDOUT, with respect to the shell process.
- **what happens when I ls?**
1. Shell _forks_ itself
2. Parent _wait_ for child
3. Child exec the LS command, passing in the program argument

## Program Arguments 
- Arguments are passed from **exec** onto the heap.
- Address and num_args are passed to the first stack frame.

## Redirection
- Change where process writes stdin, stdout to
- < *file* closes FD=0, open *file* for reading
- \> *file* closes FD=1, open *file* for writing

## Piping commandA | commandB
- STDOUT of A becomes STDIN of B.

1. Runs commandA and commandB in **parallel**
2. Shell creates a **pipe** (just a file.)
3. Shell *fork* itself twice
4. Parent shell *wait* for both children
5. One child redirects STDOUT to pipe before *exec* first command
6. Other child redirects STDIN to pipe before *exec* second command

ls -la bin ; cat notes.txt executes 1 after another.
ls -la bin | less executes at the same time.
pipelineA && pipelineB. if A returns 0, run B
pipelineA || pipelineB. if A returns !0, run B.

# Security
## Namespacing
- Provides a process with their own system view
- CGroup:
  - Each CGroup namespace has its own set of CGroup root directories
- PID: Process ID
- NET: Network
- MNT: Mount
- UTS: UNIX timesharing system
- IPC: Interprocess Communication
  - Isolates IPC resources like message queue.
- User: Separate user IDs.
  - A user may be ID=0 in their namespace, but underprivileged in an outside namespace.

## Cgroup
- Allow processes to be organized into hierarchical groups
- Usage of various types of resources can be limited and monitors
- cgroup interface is provided through the pseudo-filesystem cgroupfs.
- A *subsystem* is a kernel component that modifies the behaviour of the process in a cgroup. *subsystem* is sometimes called a resource controller.
- 

## Kernel vs User level

# LINUX troubleshooting and debugging
- when in doubt **man pages**
- _w_ for who is logged in, what are they doing.
- _lsof_ lists open files.
- TMUX: Terminal multiplexing 
  - long-running commands! we can log out and log back in.
  - Share shell sessions, 2 people logged in at the same time.
    - Share a socket file. type together in from 2 terminals.

## Check kernel version
- _uname_ prints system information
  - _uname -r_ for release
  - _uname -v_ for kernel version

## Check the hardware.
- _ethtool eth0_ tells us settings for eth0.
  - Speed, duplex, link modes.

## Manage services
- _systemctl_
- _ps_ for running processes. 
  - Look for a process with a given name:
  - ps | grep nginx

## Check the network.
- See LINUX Networking
- _ip_ is newer than _ifconfig_
1. **ip link** check if your interfaces are up.
2. **ip address** Know if your interfaces have IP addressing
  - _ip addr show eth0_ to see information on eth0.
3. **ping** Know if you have network reachability
4. **traceroute** Trace route that your network traffic takes.
- _dig www.google.com_ resolves DNS for google.com
- Check open ports?
  - **netstat -tulpn** 
  - TCP, UDP, Listening, show PID and program name, N for numeric.
- **ip route** to see route tables. (used to be **route**)
- IPTABLES. //TODO

## Check resource usage
- _top_ tells us currently running processes and CPU/memory usage of each.

## Check storage
- _df_. **Disk free**. disk space free.
  - _df -a_ for **all** file system disk spage usage
  - _df /home_ for information about /home file system.
  - _df -i_ for information about file systme inodes.
  - _df -h_ for **human readable**
- _du_. **Disk use**. Disk space usage.
  - _du -sh ./amazon_ for disk usage on ./amazon

## Check the logs.
- _/var/log/_