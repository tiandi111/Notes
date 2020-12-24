Little Endian and Big Endian
---

Endianness 
---
Little and big endian are two ways of storing multibyte data-types ( int, float, etc). In little endian machines, last
byte of binary representation of the multibyte data-type is stored first. On the other hand, in big endian machines,
first byte of binary representation of the multibyte data-type is stored first.

[Note] Only affect multi-byte data types, int, float, long, double...etc

Examples
---
```
    char a = {0x01, 0x00};
    cout<< *( (int*) a ) <<endl;
``` 
For little endian, output: 