HW1 Draft
---


Some Background
---

CPU bitness usually means the bitness of register, ALU, memory bus or data bus.

Design
---

Instructions
---
- ADD $1, $2, $3
- SUB $1, $2, $3
- MUL $1, $2, $3
- SLT $1, $2, $3    if ($2-$3) < 0  $1 = 1 else $1 = 0
- L $1, mem
- S $2, mem
- BEQ $1, $2, off
- JMP off

Test Program
--- 
Program1 Assembly:
```
L   $1, a
L   $2, b
SLT $3, $2, $1 
BEQ $3, $0, GT
ADD $1, $1, $2
JMP ST
GT:
SUB &1, $1, $2
ST:
S   $1, a
```

Program2 Assembly:
```
L   $A, a
L   $B, b
L   $C, c
LOOP1:
SLT $ri, $i, M
BEQ $ri, $0, END
add $ri, $ri, 1
LOOP2:
SLT $rj, $j, N
BEQ $rj, $0, LOOP1
ADD $rj, $rj, 1
LOOP3:
SLT $rk, $k, P
BEQ $rj, $0, LOOP2

L   $Aaddr, MEM[a]
MUL $Aaddr, $ri, N
ADD $Aaddr, $Aaddr, $k
L   $Aaddr, $Aaddr

L   $Baddr, MEM[b]
MUL $Baddr, $rk, P
ADD $Baddr, $Baddr, $rj
L   $Baddr, $Baddr

MUL $mul, $Aaddr, $Baddr
ADD $Cij, $Cij, $mul

L   $Caddr, MEM[c]
MUL $Caddr, $ri, M
ADD $Caddr, $Caddr, $rj

S   $Cij, $Caddr
ADD $k, $k, 1
JMP LOOP3
END:
```