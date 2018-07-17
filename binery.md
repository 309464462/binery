## 第一章 Linux环境和相关工具

> 二进制逆向工程的通用软件IDA Pro

### 1.1 LInux工具

#### 1.1.1 GDB
> GDB 不仅可以用来调试有BUG的应用程序，也可以用于研究甚至改变一个程序的控制流，还可以用于修改代码、寄存器和数据结构。主要用于分析ELF文件和Linux进程。

#### 1.1.2 GNU binutils中的objdump

> 一种对代码进行快速反编译的方式。起缺点是需要依赖ELF文件头，并且不会进行控制流分析。

- 查看ELF文件中的所有节的数据或代码

  ==objdump -D  <elf file>==

- 只查看ELF文件中的程序代码

  ==objdump -d  <elf file>==

- 查看所有符号

  ==objdump -tT <elf file>==

#### 1.1.3 GNU binutils中的objcopy

> 可以用来分析和修改任意类型的ELF目标文件，还可以修改ELF节，或将ELF节复制到ELF二进制中。

###### **要将.data 节从一个eldf目标文件复制到另一个elf文件中，可以使用下面**

==objcopy -only-section=.data <infile> <outfile>==

#### 1.1.4 strace

> 系统调用追踪

- 使用strace 命令来跟踪一个基本的程序：

 ==strace  /bin/ls -o ls.out==

- 使用strace 命令附加到一个现存的进程上，查看log

 ==strace -p <pid> -o demon.out==  

- 原始输出将会显示每个系统调用的文件描述符编号，系统调用会将文件描述符作为参数

  SY_read(3,buf,sizeof(buf))

  如果想看读入到文件描述符3中的所有数据，可以运行下面的命令

  ==strace -e read=3 /bin/ls==

  ==strace -e write=fd /bin/ls==

#### 1.1.5 ltrace

> 库追踪，它会分析共享库，即一个程序的链接信息，并打印出用到的库函数

#### 1.1.6 基本的ltrace命令

#### 1.1.7 ftrace

#### 1.1.8readelf

#### 1.1.9 ERESI-ELF反编译系统的接口

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     

### 1.2 有用的设备和文件

### 1.3 链接器相关环境指针

## 第二章 二进制格式
## 第三章 Linux进程追踪
## 第四章 ELF病毒技术-Linux/UNIX病毒
## 第五章 Linux 二进制保护
## 第六章 Linux下的ELF二进制取证分析
## 第七章 进程内存取证分析
## 第八章 ECFS --扩展核心文件快照技术
## 第九章 Linux/proc/kcore分析





