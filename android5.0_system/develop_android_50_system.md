### 第一章 建立Android系统开发环境

#### 1.1 安装系统

#### 1.2 安装开发包

#### 1.3 安装一些工具

#### 1.4 下载源码

> 如果实用谷歌手机，那么就应该考虑源码的版本问题，可以通过以下链接查找，需要翻墙
>
> https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds



### 第二章 Android 的编译环境Build系统

> Android 的Build系统是基于GNU Make 和 Shell 构建的一套编译环境。
>
> Android的Build系统除了完成目标系统的二进制文件，APK应用程序的编译、链接、打包等工作外，还需要生成目标文件系统的景象以及各种配置文件，同时还要维护各个模块间的依赖关系，确保某个模块的修改能引起所依赖的文件重新编译。Android build 系统的功能非常强大，能同时支持多架构（x86,arm和mips）、多语言(汇编、c/c++、java)和多目标（同时支持多个产品）

> 从大的方面，Android的Build系统可以分为三大块：
>
> 1. 位于build/core目录下的文件，这是Android Build系统的框架和核心
> 2. 位于device目录下的文件，存放着具体的产品的配置文件
> 3. 各模块的编译文件：Android.mk

2.1 Android Build系统核心

通常 编译Android系统使用下面的命令

```shell
#.build/envsetup.sh
#lunch
#make
```

2.1.1 编译环境的建立

1.  envsetup.sh 文件的作用

   > 这个脚本会建立Android的编译环境。

   ![1533536544434](D:\mybook\book_principal_work\android5.0_system\img\1533536544434.png)

  Build系统会在这里分别引入具体产品的配置文件 AndroidProduct.mk 和 BoardConfig.mk,以及各个模块的编译文件Android.mk

- clang 目录下的config.mk也会和select.mk文件一样按照同样的规则包含进3个不同的mk文件
- combo 目录下的这些 mk文件定义了GCC编译器的版本和参数。
- clang目录下的mk文件则定义了LLVM编译器的clang的路劲和参数。



### 第三章 连接Android和Linux内核的桥梁-- Android的Bionic

### 第四章 进程间通信--Android的Binder

### 第五章 连接 Java和 c/c++层的关键 ---Android的JNI

### 第六章 Android的同步和消息机制

### 第七章 第一个用户进程 -- Android的Init进程

### 第八章 支撑Android世界的一极 -- Zygote进程

### 第九章 精确地控制资源的使用 --- Android的资源管理

### 第十章 Android系统的核心之一 --SystemServer进程

### 第十一章 APK包的安装、卸载和优化-- Android的应用管理

### 第十二章 Android的组件管理

### 第十三章 Android的多用户模式

### 第十四章 Android的图形显示系统

### 第十五章Android的窗口系统

### 第十六章 Android额输入管理

### 第十七章 Android的电源管理

### 第十八章 Android的存储系统

### 第十九章 Android的网络管理框架

### 第二十章 Android的音频系统

### 第二十一章 让应用更安全---Android的SElinux模块

### 第二十二章 Dalvik和ART虚拟机

### 第二十三章 系统升级模块--- Android的Recovery模块

### 第二十四章 Android的调试方法