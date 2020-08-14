Charpter 0
---
Operation system interfaces
---

[Kernel]<br/>
A program that provides a complete set of services to applications.

Kernel resides in its own memory space, kernel space, which requires
hardware privileges to access.

[Shell]<br/>
An ordinary program that reads commands from the user and executes
them.

### Process and memory
### 1. System call
#### fork()
- Definition

Fork creates a new process, called child process, with exactly the
same memory contents as the calling process, called the parent process.

- Properties
    - duplicate memory content
    - duplicate file descriptors, but the underlying file description is shared
    - returns in both the parent and the child
        - in parent process, returns child's pid
        - in child process, returns 0
        
#### exec()
- Definition

Exec replaces the calling process's memory with a new memory image
loaded from a file stored in the file system. 

- Properties
    - when exec succeeds, it does not return to the calling program,
    instead, the instructions loaded from the file start executing at
    the entry point declared in somewhere.

#### exit()

Exit causes the calling process to stop executing and to release resources
such as memory and open files.

#### wait()
Wait returns the pid of an exited child of the current process; if none
of the caller's children has exited, wait waits for one to do so.

### 2. Shell
The xv6 shell uses the above system calls to run programs on behalf
of users.

```
        +-------+
        | main p|
        +-------+
            |
            V
   +-> while(getcmd())
   |        |
   |        V               +--------+
   |      fork() ---------> | child p|
   |        |               +--------+
   |        |                    |
   |        |                    V
   |        |                  exec()
   |        |                    |
   |        |                    V
   |        |                  exit()
   |        |                    |
   |        V                    |  
   |       wait() <--------------+
   |        |
   |        |
   +--------+
```

### I/0 and File descriptors

[File descriptor] <br/>
A file descriptor is a small integer representing a kernel-managed object that a process
can read from or write to.

The file descriptor interface abstracts away the differences between files, pipes, and
devices, making them all look like streams of bytes.

Unix-like system uses the file descriptor as an index into a per-process table, so that
every process has a private space of file descriptors starting at 0.

fd 0 -> standard output<br/>
fd 1 -> standard input<br/>
fd 2 -> standard error<br/>

### 1. System call
#### open()
- Definition
Open opens the file specified by pathname and returns a file descriptor as the reference 
to the file.

- Properties
    - if the file does not exist, it may optionally be created
    - open() returns a file descriptor
    - open() creates a new open file description (thus there may be multiple file descriptions
    corresponding to a file inode)
    
[Open file description]
The term open file description refers to an entry in the system-wide table of open files.

When a file descriptor is duplicated, the duplicate refers to the same open file description
as the original file descriptor, and the two file descriptors consequently share the same
file offset and file status flags. Such sharing can also occur between processes when a
child process is created via fork().

#### close()
- Definition
Close closes a file descriptor, so that it no longer refers to any file and may be reused.

- Properties
    - if the fd is the last descriptor referring to the underlying open file description,
    the resources associated with the open file description are freed

#### dup()
- Definition
Dup creates a copy of the file descriptor, using the lowest-numbered unused file descriptor
for the new descriptor.

- Properties
    - the oldfd and the newfd share the file offset and file status flag

### Pipes

[Pipes]<br/>
A pipe is a small kernel buffer exposed to processes as pair of file descriptors, one for
reading and one for writing. Writing data to one end of the pipe makes that data available
for reading from the other end of the pipe. 

Pipes provide a way to processes to communicate.

### Code example
```
    int p[2];
    char *argv[2];

    argv[0] = "wc";     
    argv[1] = 0;

    pipe(p); 
    if(fork() == 0) {                       // child, receiver
       close(0);        
       dup(p[0]);                           // duplicate p[0] into fd0, the stdin
       close(p[0]);                         // close old fds,
       close(p[1]);                         // note that these fds are duplcates from parent process
       exec("/bin/wc", argv);
    } else {                                // parent, sender
       close(p[0]);                         
       write(p[1], "hello world\n", 12);    // write to stdout
       close(p[1]);
    }
```

### File system