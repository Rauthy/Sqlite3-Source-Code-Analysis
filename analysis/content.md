## 文档说明

#### 前言

&emsp;&emsp;sqlite作为小型嵌入式数据库，广泛应用于手机等小型设备中，但是目前对于sqlite的分析和优化还比较少，我们希望在wal型日志的基础上进行改进并实现新的日志模式以提高其日志效率。为了达到该目的，我们查找了很多资料，在查看他人资料的同时也阅读sqlite的源码，弄清其具体实现，希望该项目能够帮助到其他同样对sqlite感兴趣的开发者。

#### 文档结构

&emsp;&emsp;分析文档的结构如下:

- [sqlite源码结构及编译过程](.\project structure and compilation.md)
- [sqlite vfs的定义](.\vfs.md)
- [rollback journal](./rollback journal.md)
- [wal](./wal.md)
- [vdbe](./vdbe.md)