Compile
---

## Tools
以下内容摘抄自https://zhuanlan.zhihu.com/p/64373941
```
总结：

gcc、clang：两个编译器，用于将程序员的编程语言，编译汇编链接成机器语言；

make：在没使用IDE时，make工具相当于一个智能的批处理工具，本身没有编译和链接的功能，而是用类似于批处理的方式通过调用makefile文件中用户指定的命令来进行编译和链接；

makefile：相当于用户将要执行的一系列命令，make根据makefile中的命令对相应的源文件进行编译和链接；

cmake：用于更加方便地生成makefile文件给make用，cmake还有其他功能，如可以跨平台生成对应平台能用的makefile，无需自己根据每个平台的不同特性去修改；

CMakeLists.txt：cmake根据CMakeLists.txt文件（组态档）去生成makefile，CMakeLists.txt可以自己写，写起来比makefile容易很多；我们使用IDE时，会自动生成各种CMakeLists.txt；
```
