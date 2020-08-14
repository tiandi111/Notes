Chapter 6
---
File system
---

Challenges the file system addresses:
- on-disk data structure to represent
    - the tree of directories and files
    - the identities of the blocks that hold each file's content
    - record which areas of the disk are free
- crash recovery
- concurrent use of multiple processes
- maintains im-memory cache of popular blocks

### xv6 file system layout
```
+-----------------+
| File descriptor | Abstracts many unix resources (e.g., pipes, devices, files, etc.) using the file system interface
+-----------------+
|     Pathname    | Hierarchical path names
+-----------------+
|    Directory    | Special inode that represent directory
+-----------------+
|      Inode      | Single file representation
+-----------------+
|     Logging     | Transaction logging, provide the atomicity of updates
+-----------------+
|   Buffer cache  | Cache disk blocks and synchronizes access to them
+-----------------+
|      Disk       | Read and write blocks on an IDE hard drive
+-----------------+
```

### xv6 file system structure
```
+------+-------+-------+------------+----------+------------------+
| boot | super |  log  |   inodes   |  bitmap  |       data       |    
+------+-------+-------+------------+----------+------------------+
   0       1    2
```
- block 0: boot

#### Buffer cache layer

#### Logging layer

