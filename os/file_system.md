File system
---

### Overview
```
+-----------------------+
|      Applications     |
+-----------------------+
| Programming libraries |
+-----------------------+
|      System calls     |
+-----------------------+
|          VFS          |
+-----------------------+
|          FSs          |
+-----------------------+
|      Block layer      |
+-----------------------+
|     Device drivers    |
+-----------------------+
|        Hardware       |
+-----------------------+
```

### VFS

Linux VFS object types:
1. inode object         ->      an individual file
2. file object          ->      an open file
3. superblock object    ->      an entire file system
4. dentry object        ->      an individual file directory

### File Implementation

The most important issue in implementing file is keeping track of which disk block goes with which
file.

#### 1. Continuous Allocation
- Pros
    - simple
    - good read performance
- Cons
    - external fragmentation
    - hard to approximate space in advance
    
#### 2. Linked List

#### 3. Linked List in Table

#### 4. I-Node
Index Node is a block that stores file-to-block addr mappings.
```
    +-----------------+
    | file attributes |
    +-----------------+
    | addr of block 1 |
    +-----------------+
    | addr of block 2 |
    +-----------------+
    |      ...        |
    +-----------------+
    |addr of block ptr| -> +--------------------------------------------+
    +-----------------+    | disk block containing addtional disk addrs |
                           +--------------------------------------------+
```

### Directory Implementation

The goal of directory system is to map the file name onto the information needed to locate
the data, no matter what kind of information it is (it could be block number, disk addr or
inode number).



### Free Space Management

### Recovery
Failure considered in this section is *system failure*

#### 1. Consistency Checking

[Consistency checker]<br/>
A system program that compares the data in the directory structure with the data blocks on disk
and tries to fix any inconsistencies it finds.

- Basic solution

    Run consistency checker for each fs.
 
- Status bit

    At the start of any metadata change, a status bit is set to indicate the change. If all updates
    to the metadata complete successfully, the file system can clear the bit. If the bit remains
    set, a consistency checker is run.
    
#### 2. Log-Structured File System

This idea comes from a common implementation of transaction in database system, journaling.
The resulting file systems are known as log-based transaction-oriented file systems, or JFS
(journaling file system).

Changes to metadata are sequentially ordered as transactions. All transactions are circular logged
first, then asynchronously applied to disk in background.

[circular logging]<br/>
Logs are written until the end of the file and then written from the beginning.

A side benefit is that metadata updates proceed faster because of the performance advantage of 
sequential I/O over random I/O.

#### 3. Other Solutions

An alternative is to always write data or metadata in a new block, when the writes are finished, the
pointer to the old blocks are redirected to the new blocks. In this way, the old blocks also serve as
snapshots.

#### 4. Backup and Restore

- Backup:
    - full backup
    - incremental backup