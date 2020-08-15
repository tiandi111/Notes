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

### Disk Space Allocation

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