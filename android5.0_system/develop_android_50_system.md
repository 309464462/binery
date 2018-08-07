---
typora-root-url: ./
---

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

#### 2.1 Android Build系统核心

通常 编译Android系统使用下面的命令

```shell
#.build/envsetup.sh
#lunch
#make
```

##### 2.1.1 编译环境的建立

1.  envsetup.sh 文件的作用

   > 这个脚本会建立Android的编译环境。它会提供一些必要的指令

```shell
- lunch:   lunch <product_name>-<build_variant>
- tapas:   tapas [<App1> <App2> ...] [arm|x86|mips|armv5|arm64|x86_64|mips64] [eng|userdebug|user]
- croot:   Changes directory to the top of the tree.
- m:       Makes from the top of the tree.
- mm:      Builds all of the modules in the current directory, but not their dependencies.
- mmm:     Builds all of the modules in the supplied directories, but not their dependencies.
           To limit the modules being built use the syntax: mmm dir/:target1,target2.
- mma:     Builds all of the modules in the current directory, and their dependencies.
- mmma:    Builds all of the modules in the supplied directories, and their dependencies.
- cgrep:   Greps on all local C/C++ files.
- ggrep:   Greps on all local Gradle files.
- jgrep:   Greps on all local Java files.
- resgrep: Greps on all local res/*.xml files.
- sgrep:   Greps on all local source files.
- godir:   Go to th
```

先看实际执行部分的代码

```shell
# add the default one here
add_lunch_combo aosp_arm-eng
add_lunch_combo aosp_arm64-eng
add_lunch_combo aosp_mips-eng
add_lunch_combo aosp_mips64-eng
add_lunch_combo aosp_x86-eng
add_lunch_combo aosp_x86_64-eng

#结尾处，
# Execute the contents of any vendorsetup.sh files we can find.
#表示搜索所有vender device目录的 vendorsetup.sh 
for f in `test -d device && find -L device -maxdepth 4 -name 'vendorsetup.sh' 2> /dev/null` \
         `test -d vendor && find -L vendor -maxdepth 4 -name 'vendorsetup.sh' 2> /dev/null`
do
    echo "including $f"
    #运行这些vendorsetup.sh文件
    . $f
done
unset f
```

以华为的device/lge/hammerhead 设备为例

查看vedorsetup.sh可以看到只有一句话，添加到变量中

```shell
add_lunch_combo aosp_hammerhead-userdebug
```

原函数为

```shell
unset LUNCH_MENU_CHOICES 
##取消设置一个shell变量，从内存和shell的导出环境中删除它
#shell支持一维数组（不支持多维数组），并且没有限定数组的大小。
# 将调用该命令所传递参数存到一个全局的数组变量LUNCH_MENU_CHOICES
function add_lunch_combo()
{
    local new_combo=$1
    local c
    ##使用@ 或 * 可以获取数组中的所有元素，
    for c in ${LUNCH_MENU_CHOICES[@]} ; do
        if [ "$new_combo" = "$c" ] ; then
            return
        fi
    done
    LUNCH_MENU_CHOICES=(${LUNCH_MENU_CHOICES[@]} $new_combo)
}
```

![1533469089834](E:\mybook\book_principal_work\android5.0_system\ing\1533469089834.png)

2. Lunch命令功能

##### 2.1.3 Build系统的层次关系

> 设置好变量之后，接下来的make命令你就会开始执行编译工作。 make命令会调用 build目录下的Makefile文件，它的内容是：

```
include build/core/main.mk
```

makefile 主要包含了3种内容： 变量定义/函数定义/目标依赖规则。

![](E:\mybook\book_principal_work\android5.0_system\img\1533536544434.png)

  Build系统会在这里分别引入具体产品的配置文件 AndroidProduct.mk 和 BoardConfig.mk,以及各个模块的编译文件Android.mk

- clang 目录下的config.mk也会和select.mk文件一样按照同样的规则包含进3个不同的mk文件
- combo 目录下的这些 mk文件定义了GCC编译器的版本和参数。
- clang目录下的mk文件则定义了LLVM编译器的clang的路劲和参数。

![1533622320041](/img/1533622320041.png)

##### 2.1.4 分析main文件

 main.mk文件是Android Build系统的主控文件，分段解释如下.

(1) 检查 gnu make的版本是否大于或等于3.8.1，否则报错并停止编译：

```shell
ifeq (,$(findstring CYGWIN,$(shell uname -sm)))  //判断如果不是window版本
ifneq (1,$(strip $(shell expr $(MAKE_VERSION) \>= 3.81))) 
$(warning ********************************************************************************)
$(warning *  You are using version $(MAKE_VERSION) of make.)
$(warning *  Android can only be built by versions 3.81 and higher.)
$(warning *  see https://source.android.com/source/download.html)
$(warning ********************************************************************************)
$(error stopping)
endif
endif

```

(2) 定义缺省的编译目标为“droid”。因此，命令“make" 相当于”make droid“:

```shell
# This is the default target.  It must be the first declared target.
.PHONY: droid
DEFAULT_GOAL := droid
$(DEFAULT_GOAL):
```

(3) 引入几个make文件。注意“-include”和“include” 的区别是：前者包含的文件如果不存在不会报错，后者则会报错并停止编译。

```shell
# Targets that provide quick help on the build system.
include $(BUILD_SYSTEM)/help.mk

# Set up various standard variables based on configuration
# and host information.
include $(BUILD_SYSTEM)/config.mk

# This allows us to force a clean build - included after the config.mk
# environment setup is done, but before we generate any dependencies.  This
# file does the rm -rf inline so the deps which are all done below will
# be generated correctly
include $(BUILD_SYSTEM)/cleanbuild.mk

# Include the google-specific config
-include vendor/google/build/config.mk

VERSION_CHECK_SEQUENCE_NUMBER := 5
-include $(OUT_DIR)/versions_checked.mk
```

(4) 检查java的版本是否是1.7或者1.6 ，不是则会报错退出。如果java版本是1.7，在linux下要求必须是openjdk的版本，否则要求是Oracle的JDK版本：

```shell
java_version_str := $(shell unset _JAVA_OPTIONS && java -version 2>&1)
javac_version_str := $(shell unset _JAVA_OPTIONS && javac -version 2>&1)

# Check for the correct version of java, should be 1.7 by
# default, and 1.6 if LEGACY_USE_JAVA6 is set.
ifeq ($(LEGACY_USE_JAVA6),)
required_version := "1.7.x"
required_javac_version := "1.7"
java_version := $(shell echo '$(java_version_str)' | grep '^java .*[ "]1\.7[\. "$$]')
javac_version := $(shell echo '$(javac_version_str)' | grep '[ "]1\.7[\. "$$]')
else # if LEGACY_USE_JAVA6
required_version := "1.6.x"
required_javac_version := "1.6"
java_version := $(shell echo '$(java_version_str)' | grep '^java .*[ "]1\.6[\. "$$]')
javac_version := $(shell echo '$(javac_version_str)' | grep '[ "]1\.6[\. "$$]')
endif # if LEGACY_USE_JAVA6

ifeq ($(strip $(java_version)),)
$(info ************************************************************)
$(info You are attempting to build with the incorrect version)
$(info of java.)
$(info $(space))
$(info Your version is: $(java_version_str).)
$(info The required version is: $(required_version))
$(info $(space))
$(info Please follow the machine setup instructions at)
$(info $(space)$(space)$(space)$(space)https://source.android.com/source/initializing.html)
$(info ************************************************************)
$(error stop)
endif

# Check for the current JDK.
#
# For Java 1.7, we require OpenJDK on linux and Oracle JDK on Mac OS.
# For Java 1.6, we require Oracle for all host OSes.
requires_openjdk := false
ifeq ($(LEGACY_USE_JAVA6),)
ifeq ($(HOST_OS), linux)
requires_openjdk := true
endif
endif
```



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