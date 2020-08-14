OS memory layout
---

```
+-----------------------+  -> High Address
| Command Line Args &   |
| Environment Variables |
+-----------------------+
|         STACK         |
|           ↓           |
+-----------------------+
|           ↑           | 
|          HEAP         |
+-----------------------+
|          BSS          |
|   Uninitialized data  |
+-----------------------+
|          DATA         |
|    Initialized data   |
+-----------------------+
|          TEXT         |
+-----------------------+  -> Low Address
```

- TEXT:  instructions
- Data:  initialized global variables
- BSS:   uninitialized global variables (stand for "block started by symbols")
- Heap:  dynamic memory allocated
- Stack: program stack
