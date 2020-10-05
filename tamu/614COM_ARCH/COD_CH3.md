Computer Organization and Desgin
---
Chapter 3 | Arithmetic for Computers
---

### 3.5 Floating Point
- floating-point represent number in scientific notation
- design compromise: fraction vs exponent
    - fraction -> precision
    - exponent -> range   
- format
    - s: sign
    - fraction: [0, 1), will be added with 1
```
+----+-----------+---------------+
| 31 | 30 ... 20 | 19 18 ... 1 0 |
+----+-----------+---------------+
| s  |  fraction |    exponent   |  
+----+-----------+---------------+
fp = (-1)^s * (1 + fraction) * 2^exp
```