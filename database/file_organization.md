Database System Concepts, Ch13 Data Storage Structures
---

### File Organization
- Fixed-length Records
    - problem1: internal fragmentation
        - the space records take cannot always perfectly fit in a page
    - problem2: deletion of records
        - we do not want to move records after deletion because disk access is expensive
        - we use a 'freelist' to indicate free space holes inside the file, new records will be inserted 
        directly to entries in freelist 
- Variable-length Records
    - problem1: how to represent records in file so that attributes can be extracted easily
        - variable-length attributes are represented in the initial part of the record by a pair(offset, length)
    - problem2: how to store variable-length records so that each record can be extracted easily
        - slotted-page structure:
            - header: the number of record entries, the end of free space in the block, an array whose entries
            contain the location and size of each record
            - records: are allocated continuously in the block starting from the end of the block
            - insertion: allocated from the end of free space 
            - deletion: entry in the header is set to deleted, the records are compacted to fill in the hole leaved
            by the deletion

### Organization of Records in Files
- Heap File Organization <br/>
    Records have no ordering in files.
    - if there are no free space holes, new records are appended at the end of the files
    - if there are free space holes, use free-space map to find one that has sufficient space
        - use 1 byte to indicate the free space percent, e.g, 10000000 means that a half of space is free (128/256)
        - use multi-level free-space map to speed up the searching process (e.g, a second level to indicate the largest
        free space hole in the first 100 blocks)
            
- Sequential File Organization <br/>
    Records are stored sequentially in files.

- Multitable Clustering File Organization <br/>
    Records from different tables are stored in a single file to improve efficiency on certain join operations.
    
- B+tree File Organization
    Records are organized in a b+tree to speed up insertion, deletion, update and query. B+tree Index and records
    are stored together, also called clustering index.

- Hashing File Organization

### Data-Dictionary Storage
The Data-Dictionary (also called system catalog) stores metadata; that is, data about data,
such as:
- 

### Buffer Manager (Virtual memory manager for Database)
Database system seeks to minimize the number of blocks transfers between disk and
memory. We can reduce the number of disk accesses by keeping as many blocks as
possible in main memory.
- How Buffer Manager works?
    Program call on the buffer manager when they need a disk block.
    - If the block is in buffer, the buffer manager return its address
    - If the block is not in buffer
        - Allocate space in the buffer for the block
            - Replacing some blocks to make space for the new block if required
            - Replaced block written back to disk only if it was modified since 
            the most recent time that is was written to/fetched from the disk
        - Read the block from disk into the buffer, and return the address of it 
        in the buffer
- Buffer Replacement policies