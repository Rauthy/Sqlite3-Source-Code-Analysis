## sqlite源码结构及编译

#### 简介

&emsp;&emsp;sqlite的源码可以在[sqlite官网](https://www.sqlite.org/index.html)以及对应的[github](https://github.com/sqlite/sqlite)仓库找到，下载下来以后就可以到本地编译运行了，也可以查看相关源码进行分析。

#### 项目结构

&emsp;&emsp;编译前项目的文件结构如下：

<img src="..\image\project structure before compilation.png" alt="project structure before compilation" style="zoom:80%;" />

&emsp;&emsp;几个重要的部分如下:

- **tool**   该目录下主要包含了生成sqlite源码需要tcl脚本程序

  - *mksqlite3c.tcl*   用来生成sqlite3.c文件，基本是将src及ext文件夹中的源文件合并而来
  - *mksqlite3h.tcl*  用来生成sqlite3.h文件
  - *mkshellc.tcl*  用来生成shell.c文件

  sqlite3.c包含了sqlite主要功能的实现，sqlite3.h是主要的头文件，定义了相关的变量及接口，shell.c是sqlite的命令行程序，如果需要将sqlite编译成命令行程序，将三者放在同一文件夹下，并运行一下编译命令即可得到相应的sqlite命令行程序

  ```shell
  gcc -o  sqlite shell.c sqlite3.c  -ldl -lz -lpthread 
  ```

- **test**   test文件夹下是sqlite的测试文件

- **src**   src是sqlite主要的源码目录

- **ext**   ext是sqlite扩展功能的源码目录，例如fts，rtree等

#### 编译

&emsp;&emsp;一般使用linux版本的代码进行分析，在linux上编译需要安装tcl(tool command language)工具，[下载地址](http://www.tcl.tk/software/tcltk/download.html)，所以我在ubuntu16上编译了源码，具体过程如下：

- 解压源码
- 运行文件夹里头的configure命令，./configure
- 运行make命令

我是在虚拟机的ubuntu16上编译的，为了阅读方便，我将编译后的文件目录拷贝到了本机上。编译完以后的文件目录如下：

![编译后的文件夹](..\image\after compilation.png)

与源码文件相比，编译还产生了如下文件：

- **sqlite3.c**   编译过程会将与sqlite3相关的文件整合为一个文件sqlite3.c，官方的说法是amalgamation，这样如果要在sqlite基础上进行开发也比较方便，不过缺点是文件很大，不便于调试，官方提供了tcl脚本将该文件拆分为多个小文件
- **sqlite3.h** 这个文件包含了sqlite3的函数及变量的声明
- **./tsrc** 这个文件夹包含了sqlite3的源码文件，基本是拷贝自src，可以用来分析源码实现

以上只是部分总结，还可以利用tcl脚本命令运行测试相关的代码。

sqlite的部分源码文件很大(5000行左右)，全部看完不太可能，可以结合编辑器的查找命令查看相关函数及变量的定义，幸运的是，sqlite的代码注释非常丰富，可读性非常高，想详细了解的话可以阅读源码及对应注释。为了平台的兼容，sqlite3在工程上做了很多设计。由于时间有限，我只了解项目的大体结构及部分核心实现，不太重要的部分则快速带过。详细的源码功能分析可以查看[源码分析](http://huili.github.io/sqlite/system.html)

