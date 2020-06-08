Cache vs Buffer
---
Yet another pair of confusing concepts.

###What is?

- Read Cache

    Read Cache temporarily store the data in some kind of fast storage so that 
    the subsequent reads can be fulfilled from that storage with a faster speed.

- Read Buffer

    Read Buffer is read only when it is full. For small amount of data, this can
    reduce the total times to read from storage. For large amount of data, this 
    makes the reading curve smoother.

- Write Cache

    Write Cache temporarily store the data that will be written so that multiple
    identical writes will only be executed once

- Write Buffer

    Write Buffer is used to temporarily store the data that will be written to 
    storage. Write is only trigger when the buffer is full. Again, for small amount
    of data, this reduces the total times of writes. For large amount of data, this
    makes the writing curve smoother.