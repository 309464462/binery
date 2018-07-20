---
typora-root-url: img
---

### 第一章 Linux环境和相关工具

> 二进制逆向工程的通用软件IDA Pro

#### 1.1 LInux工具

##### 1.1.1 GDB
> GDB 不仅可以用来调试有BUG的应用程序，也可以用于研究甚至改变一个程序的控制流，还可以用于修改代码、寄存器和数据结构。主要用于分析ELF文件和Linux进程。

##### 1.1.2 GNU binutils中的objdump

> 一种对代码进行快速反编译的方式。起缺点是需要依赖ELF文件头，并且不会进行控制流分析。

- 查看ELF文件中的所有节的数据或代码

  ==objdump -D  <elf file>==

- 只查看ELF文件中的程序代码

  ==objdump -d  <elf file>==

- 查看所有符号

  ==objdump -tT <elf file>==

##### 1.1.3 GNU binutils中的objcopy

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

##### 1.1.5 ltrace

> 库追踪，它会分析共享库，即一个程序的链接信息，并打印出用到的库函数

##### 1.1.6 基本的ltrace命令
> 除了可以查看库函数调用之外，还可以使用-S 标记查看系统调用。ltrace命令通过解析可执行文件的动态段，并打印出共享库和静态库的实际符号和函数，来提供更细粒度的信息

   ==ltrace <program> -o program.out  //针对运行的程序

![微信图片_20180717223634](E:\mybook\binery\img\微信图片_20180717223634.png)

##### 1.1.7 ftrace

> 作者自己写的 在https://github.com/elfmaster/ftrace

##### 1.1.8readelf

> 在进行反编译之前，需要收集目标文件相关的信息，该命令能够提供收集信息所需要的特定于elf的所有数据。

- 查询节表头

  readelf -S <object>

- 查询程序头表

  readelf -l <object>

- 查询符号表

  readelf -s <object>

- 查询ELF文件头数据

  readelf -n <object>

- 查询重定位入口

  readelf -r <object>

- 查询动态段

  readelf -d <object>



##### 1.1.9 ERESI-ELF反编译系统的接口

> (http://www.eresi-project.org) 中含有许多linux二进制工具                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                

#### 1.2 有用的设备和文件

> /proc入口对于黑客是很有用的

##### 1.2.1 /proc/<pid>/maps

> 这个文件保存了一个进程镜像的布局，通过展现每个内存映射来实现，展现的内容包括可执行文件，共享库、栈、堆和VDSO等。

![微信图片_20180717223653](/微信图片_20180717223653.png)

![微信图片_20180717223718](/微信图片_20180717223718.png)

#### 1.2.2 /proc/kcore

> 内核的动态核心文件
>
> GDB可以使用/proc/kcore来对内核进行调试和分析。

##### 1.2.3 /boot/System.map

> 包含了整个内核的所有符号

![微信图片_20180717223750](/微信图片_20180717223750.png)

##### 1.2.4 /proc/kallsyms

> 和System.map类似，其可以动态更新，如果动态安装内核模块，符号会自动添加到/proc/kallsyms中

##### 1.2.5 /proc/iomem

> 与/proc/<pid>/maps类似，不过他是跟系统内存相关的。例如想知道内核的text段所映射的物理内存位置，可以搜索kernel字符串，然后可以查看code/text段 、data段和bss段的相关内容

![微信图片_20180717223550](/微信图片_20180717223550.png)

##### 1.2.6 ECFS

> (extened core file snapshot) 扩展核心文件快照 是一项特殊的核心转储技术，专门为进程镜像的高级取证分析所设计。

#### 1.3 链接器相关环境指针

> 动态加载器/链接器以及链接的概念，在程序链接、执行的过程中都是避不开的基本组成部分。需要特别理解：链接、重定向、动态加载的过程

##### 1.3.1 LD_PRELOAD环境变量

> 可以设置成一个指定库的路径，动态链接时候可以比其他的库有更高的优先级。这就允许预加载苦衷的函数和符号能够覆盖掉后续链接的库中的函数和符号。这在本质上就允许你通过重定向共享库函数来进行运行时修复。

##### 1.3.2 LD_SHOW_AUXV环境变量

> 这个环境变量可以通知程序加载器来展示程序运行时的辅助向量。辅助向量是放在程序栈上的信息。例如：想要获取进程镜像VDSO页的内存地址（也可以通过maps文件获取），就需要查询AT_SYSINFO

![微信图片_20180717225605](/微信图片_20180717225605.png)



##### 1.3.3 链接器脚本

> 链接器脚本是由链接器解释的，把程序划分为相应的节、内存和符号。默认链接器脚本可以使用
>
> ==ld -verbose==   查看 版本和具体内容

![微信图片_20180717230233](/微信图片_20180717230233.png)



gcc  可以通过 -T 标志来指定链接脚本

### 第二章 二进制格式

> 将会涉及到 ELF文件类型、程序头、节头、符号、重定位、动态链接、编码ELF解析器

#### 2.1 ELF文件类型

一个ELF文件可以标记为以下几种类型之一

- ET_NONE:未知类型。
- ET_REL:重定位文件。ELF类型标记为relocatable 意味着该文件被标记为了一段可重定位的代码，有时也称目标文件。可重定位目标文件通常是还未被链接到可执行程序中的一段位置独立的代码（也就是解析阶段的文件）。在编译完代码之后通常可以看到一个.o格式的文件，这种文件包含了创建可执行文件所需要的代码和数据。
- ET_EXEC:可执行文件。ELF类型为executable，表明这个文件标记为可执行文件。这种类型的文件也称为程序，是一个进程开始执行的入口。
- ET_DYN：共享目录文件。ELF类型为dynamic，意味着该文件被标记为了一个动态的可链接的目标文件，也称为 共享库。这种共享库会在程序运行时被转载并链接到程序的进程镜像中。
- ET_CORE:核心文件。在程序崩溃或者进程传递了一个SIGSEGV信号（分段违规，一般是内存违规访问导致的）时，会在核心文件中记录整个进程的镜像信息。可以使用GDB读取这类文件来辅助调试并查找程序崩溃的原因。

使用readelf  -h 命令查看ELF文件，可以看到原始的ELF文件头。ELF文件头从文件的0偏移量开始，是 除了文件头之后剩余部分文件的一个映射。

```c
 typedef struct {
               unsigned char e_ident[EI_NIDENT];
               uint16_t      e_type;
               uint16_t      e_machine;
               uint32_t      e_version;
               ElfN_Addr     e_entry;
               ElfN_Off      e_phoff;
               ElfN_Off      e_shoff;
               uint32_t      e_flags;
               uint16_t      e_ehsize;
               uint16_t      e_phentsize;
               uint16_t      e_phnum;
               uint16_t      e_shentsize;
               uint16_t      e_shnum;
               uint16_t      e_shstrndx;
           } ElfN_Ehdr;

```

#### 2.2 ELF 程序头

> 段是在内核转载是被解析的，描述了磁盘上可执行文件的内存布局以及如何映射到内存中。可以通过引用原始ELF头中名为==e_phoff==程序头偏移量来得到程序头表。
>
> 接下来介绍5中常见的elf程序头类型



> 首先看下 Elf32_Phdr/Elf64_Phdr结构体

```c
		typedef struct {
               uint32_t   p_type;
               Elf32_Off  p_offset;
               Elf32_Addr p_vaddr;
               Elf32_Addr p_paddr;
               uint32_t   p_filesz;
               uint32_t   p_memsz;
               uint32_t   p_flags;
               uint32_t   p_align;
           } Elf32_Phdr;

           typedef struct {
               uint32_t   p_type;
               uint32_t   p_flags;
               Elf64_Off  p_offset;
               Elf64_Addr p_vaddr;
               Elf64_Addr p_paddr;
               uint64_t   p_filesz;
               uint64_t   p_memsz;
               uint64_t   p_align;
           } Elf64_Phdr;

```



##### 2.2.1 PT_LOAD

> 一个可执行文件至少有一个PT_LOAD类型的段。这类程序头描述的是可装载的段，也就是，这种类型的段将被装载或者映射到内存中。

> 一个需要动态链接的ELF可执行文件通常包含了以下两个可装载的段（类型为PT_LOAD）:
>
> - 存放程序代码的text段
> - 存放全局变量的动态链接信息的data段

### 第三章 Linux进程追踪
### 第四章 ELF病毒技术-Linux/UNIX病毒
### 第五章 Linux 二进制保护
### 第六章 Linux下的ELF二进制取证分析
### 第七章 进程内存取证分析
### 第八章 ECFS --扩展核心文件快照技术
### 第九章 Linux/proc/kcore分析

### 

### 

