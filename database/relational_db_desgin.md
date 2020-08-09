Relational Database Design
---

Design goal:
- no redundancy
- easy to retrieve information

Major pitfalls:
- redundancy (DRY)
- incompleteness

Primary key vs Candidate key
- primary key is unique key and non-null, there can be only one primary key
- candidate key is unique key and there can be multiple candidate keys
    - candidate key include primary key
    - candidate key can has null column

Normal form:
- NF1: all attributes are atomic属性不可再分
- NF2: each attribute must either belongs to a candidate key or completely depend on all candidate keys
每个属性要么属于候选键要么完全依赖候选键，不能有部分依赖的情况发生
- NF3: each attribute must either belongs to a candidate key or only completely depend on candidate keys, in
other words, attributes cannot depend on others that is not a candidate key每个属性要么属于候选键要么只依赖于候选键