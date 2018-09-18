序言

> “这问题没遇到过啊” -> 测试不全面
>
> “不应该这样啊”  ->  逻辑设计有问题
>
> "这么会出现这种情况" -> 考虑不周全
>
> 别人反馈的时候，无论有多少话想说，先放一边去，静心倾听



> ##### 目标  -> 问题 ->  分析  ->解决

## 第一部分 版本信息管理

### 1 关于版本

#### 1.1 版本的编号

##### 1.1.1  软件版本的一般编号规则

格式为：项目阶段.量产进度.修改次数 

项目阶段：0 表示研发阶段，1 表示量产阶段，大版本改动+1

量产阶段：0-87表示量产前   87-n 表示量产中，每次发布+1

修改次数:  自己内部修改



例如 

==#define ECHOVOICE_VERSION "0.91.1.wakeup:iriver,ioctl test"==





## 第二部分  关于调试

### 第2章 常用的调试方式

#### 2.1 app的调试

#### 2.1.1 logcat

- 使用android studio查看
- 试用adb logcat [-s  tags] 查看

#### 2.2内核的调试

>  可以查看系统相关的消息

#### 2.2.1 dmesg

例如查看 往input子系统write数据时候，系统的消息分发。可以通过demsg查看

2.3 查看二进制源码

2.3.1 查看汇编对应的代码

> ./prebuilts/gcc/linux-x86/arm/arm-eabi-4.7/bin/arm-eabi-objdump -S out/dvb > dvb_map.txt

2.3.2 查看具体地址的代码

> prebuilts/gcc/linux-x86/arm/arm-eabi-4.7/bin/arm-eabi-addr2line -f -e out/dvb 22020 

2.3 android studio查看系统源码

利用AndroidStudio查看android源码，一般需要源码编译完成后生成  out/host/linux-x86/framework/idegen.jar ,该jar包是用于生成IDE工程中相关文件的工具。我们可以下载别人编译好的该jar包，放到相应路径下进行使用。具体操作步骤：

1、下载idegen.jar（或者编译源码也可以生成），放到out/host/linux-x86/framework/  （如果路径不存在，新建）。

2、在源码根目录下 运行 development/tools/idegen/idegen.sh 脚本。会在源码根目录下生成 android.iml、android.ipr两个文件。

3、用androidStudio代开 android.ipr即可。



注：

在idegen.sh 脚本中实际运行的是 java -cp $idegenjar Main 命令。所以需要有idegen.jar。从名称也可以看出作用 IDE generate

```
https://pan.baidu.com/s/1Pt78LD4SHiYXBQ1BQGJhYg
```



## 第三部分 异常处理

第3章  

3.1 处理所有与内存处理相关系统函数的异常。

3.1.1 mem***之类的接口

3.1.2 str***之类的接口

1. strstr()

> strstr（）参数有null的情况，当这种情况出现时，就会报段错误的信息。所以strstr（）对参数要求还是比较严格的，用的时候注意这一样。 

3.1.3 需要fork子进程的接口，比如 system()

3.1.4 文件处理过程中，文件路径是否存在等异常处理

3.1.5 所有内存处理相关的函数可能导致的异常问题

3.2 危险的脚本命令

3.2.1 rm

这个脚本不要在频繁调度的程序里面使用，用可能会导致内存异常，破坏程序的稳定性

## 第四部分 产品量产

第4章 

1.所有认证都要尽量在认证期的前期完成。最好用一半时间就可以

在交给认证之前，必须整理好硬件的调试文档，确保认证机构可以按照文档一步到位完成所有测试。



第 5 章 硬件上的问题

5.1 硬件一定要尽快功能模块，对样品的性能一定提前评估。

对功能模块的的用量，需要在工程开始之初的确定了后，采购的物料不能一开始就定下来。

需要先寻找部分样品进行稳定性评估。



第五部分

第六章

6.1 常用查找工具

6.1.1 find

6.1.3 grep