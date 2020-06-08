Transaction(intro)
---

ACID
---
- Atomicity: either all succeed or none succeeds
- Consistency: from one valid state to another valid state
- Isolation: concurrent executions does not affect the result
- Durability: data is persistent once the transaction is committed

Implementation
---
- Write-ahead log (WAL):
    - a family of techniques used to ensure atomicity and durability
    - the changes must be recorded onto stable storage before written to the database
    - in-place, reduce the need to modify indexes and block lists; (disadvantages?)
    - examples: ARIES
    - variants: journaling
- Shadow paging:
    - a technique used to ensure atomicity and durability
    - a copy-on-write technique to avoid in-place updates of pages
    - modify data on shadow pages, and when commit, all references to the old pages are redirect to
      the shadow pages
    - not in-place, (advantages?); if the referring page must also be updated via shadow paging, 
    the procedure may recurse many times, becoming quite costly
    
Concurrency Control
---
- Locking:
    - always acquire lock before modifying data
    - readers and writers block each other
    
- Multiversion concurrency control (MVCC):
    - readers get the prior version of the data that is being modified by writers, though may be
    out-of-date, the data is consistent to readers
    - readers and writers does not block each oterh
    - example: snapshot isolation
    
Distributed Transaction
---

    
    