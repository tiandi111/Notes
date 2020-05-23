My Code Checklist
---

1.检查命名是否规范，是否统一，是否直观可读<br/>
2.检查是否存在资源泄漏
- 内存泄漏
    - 清空指针，常见的如删除链表中的结点时，应当同时清空该结点的指针
- goroutine泄漏
- channel泄漏

3.interface判空问题
- interface只有在type和value都为nil时才可用==nil判空
- 