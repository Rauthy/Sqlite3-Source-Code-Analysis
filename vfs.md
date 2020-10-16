## VFS层

#### 作用 

&emsp;&emsp;VFS即virtual file system，对应sqlite七层结构中的最底层os接口部分。主要是为了保证各平台间的可移植性。

![结构](E:\数据库\sqlite\sqlite\源码阅读\图片\结构.gif)

这一层相对比较简单，主要是封装了os中的文件操作，并对上提供统一的接口实现，包括数据库文件的