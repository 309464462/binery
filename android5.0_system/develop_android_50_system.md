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

> findstring In ,STR : 在str中查找IN，如果存在就返回IN
>
> strip :  为二进制文件去掉符号信息，只要结果
>
> expr $(MAKE_VERSION) \>= 3.81  //反斜杠是为避免被shell误解
>
> 

(2) 定义缺省的编译目标为“droid”。因此，命令“make" 相当于”make droid“:

```shell
PWD := $(shell pwd)

TOP := .
TOPDIR :=

BUILD_SYSTEM := $(TOPDIR)build/core
# This is the default target.  It must be the first declared target.
.PHONY: droid
DEFAULT_GOAL := droid
$(DEFAULT_GOAL):
```

> 拿clean举例，如果make完成后，自己另外定义一个名叫clean的文件，再执行make clean时，将不会执行rm命令。  　　
>
> 为了避免出现这个问题，需要.PHONY: clean 

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

> $0		当前脚本的文件名
>
> $n		传递给脚本或函数的参数。n 是数字，表示第几个参数。例如，第一个是$1，第二个是$2。
>
> $#		传递给脚本或函数的参数个数。
>
> $*		传递给脚本或函数的所有参数。
>
> $@		传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。
>
> $?		上个命令的退出状态，或函数的返回值。一般情况下，大部分命令执行成功会返回 0，失败返回 1。
>
> $$		当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。
>
> $* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。
>
> 但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。
>
> $( ) 与 &apos;  (反引号) 都是用来做命令替换用 
>
>  file=/dir1/dir2/dir3/my.file.txt 
>
>  ${ } 分别替换获得不同的值： 
>
> ${file#*/}：拿掉第一条 / 及其左边的字符串dir1/dir2/dir3/my.file.txt*
>
> ${file##*/}：拿掉最后一条 / 及其左边的字符串：my.file.txt 
>
> ${file#*.}：拿掉第一个 .  及其左边的字符串：file.txt 
>
> ${file##*.}：拿掉最后一个 .  及其左边的字符串：txt 
>
> ${file%/*}：拿掉最后条 / 及其右边的字符串：/dir1/dir2/dir3  
>
> ${file%%/*}：拿掉第一条 / 及其右边的字符串：(空值) 
>
> ${file%.*}：拿掉最后一个 .  及其右边的字符串：/dir1/dir2/dir3/my.file 
>
> *${file%%.*}：拿掉第一个 .  及其右边的字符串：/dir1/dir2/dir3/my 
>
>  

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

> '='就是赋值运算
>
>  ':='就是当冒号前面的变量不存在或值为空时，就把等号后的值赋值给变量 
>
> '+='这个应该不用解释吧，和C中一样，变量等于本身和另一个变量的和
>
>  '?='它的意思是在语句 a?b 中如果a未定义则用b替换a 

（5）将变量 VERSION_CHECKED和BUILD_EMULATOR写入文件 out/version_checked.mk。 下次build的时候就会重新包含这个文件：

```shell
ifndef BUILD_EMULATOR
  # Emulator binaries are now provided under prebuilts/android-emulator/
  BUILD_EMULATOR := false
endif

$(shell echo 'VERSIONS_CHECKED := $(VERSION_CHECK_SEQUENCE_NUMBER)' \
        > $(OUT_DIR)/versions_checked.mk)
$(shell echo 'BUILD_EMULATOR ?= $(BUILD_EMULATOR)' \
        >> $(OUT_DIR)/versions_checked.mk)
endif

```

（6）再包含3个文件

```shell

# Bring in standard build system definitions.
include $(BUILD_SYSTEM)/definitions.mk

# Bring in dex_preopt.mk
include $(BUILD_SYSTEM)/dex_preopt.mk

ifneq ($(filter user userdebug eng,$(MAKECMDGOALS)),)
$(info ***************************************************************)
$(info ***************************************************************)
$(info Do not pass '$(filter user userdebug eng,$(MAKECMDGOALS))' on \
       the make command line.)
$(info Set TARGET_BUILD_VARIANT in buildspec.mk, or use lunch or)
$(info choosecombo.)
$(info ***************************************************************)
$(info ***************************************************************)
$(error stopping)
endif

ifneq ($(filter-out $(INTERNAL_VALID_VARIANTS),$(TARGET_BUILD_VARIANT)),)
$(info ***************************************************************)
$(info ***************************************************************)
$(info Invalid variant: $(TARGET_BUILD_VARIANT)
$(info Valid values are: $(INTERNAL_VALID_VARIANTS)
$(info ***************************************************************)
$(info ***************************************************************)
$(error stopping)
endif

# -----------------------------------------------------------------
# Variable to check java support level inside PDK build.
# Not necessary if the components is not in PDK.
# not defined : not supported
# "sdk" : sdk API only
# "platform" : platform API supproted
TARGET_BUILD_JAVA_SUPPORT_LEVEL := platform

# -----------------------------------------------------------------
# The pdk (Platform Development Kit) build
include build/core/pdk_config.mk
```

(7) 如果变量ONE_SHOT_MAKEFILE的值不为空，将它定义的文件包含进来。当编一个单独模块，ONE_SHOT_MAKEFILE 的值会设为模块的make文件路径。如果值ONE_SHOT_MAKEFILE 为空，说明再编译整个文件系统，因此，调用findleaves.py 脚本搜索系统里所有的Android.mk文件并将它们包含进来。

```shell

ifneq ($(ONE_SHOT_MAKEFILE),)
# We've probably been invoked by the "mm" shell function
# with a subdirectory's makefile.
include $(ONE_SHOT_MAKEFILE)
# Change CUSTOM_MODULES to include only modules that were
# defined by this makefile; this will install all of those
# modules as a side-effect.  Do this after including ONE_SHOT_MAKEFILE
# so that the modules will be installed in the same place they
# would have been with a normal make.
CUSTOM_MODULES := $(sort $(call get-tagged-modules,$(ALL_MODULE_TAGS)))
FULL_BUILD :=
# Stub out the notice targets, which probably aren't defined
# when using ONE_SHOT_MAKEFILE.
NOTICE-HOST-%: ;
NOTICE-TARGET-%: ;

# A helper goal printing out install paths
.PHONY: GET-INSTALL-PATH
GET-INSTALL-PATH:
	@$(foreach m, $(ALL_MODULES), $(if $(ALL_MODULES.$(m).INSTALLED), \
		echo 'INSTALL-PATH: $(m) $(ALL_MODULES.$(m).INSTALLED)';))

else # ONE_SHOT_MAKEFILE

ifneq ($(dont_bother),true)
#
# Include all of the makefiles in the system
#

# Can't use first-makefiles-under here because
# --mindepth=2 makes the prunes not work.
subdir_makefiles := \
	$(shell build/tools/findleaves.py --prune=$(OUT_DIR) --prune=.repo --prune=.git $(subdirs) Android.mk)

$(foreach mk, $(subdir_makefiles), $(info including $(mk) ...)$(eval include $(mk)))

endif # dont_bother

endif # ONE_SHOT_MAKEFILE
```

> eval 的作用是再次执行命令行处理,也就是说,对一个命令行,执行两次命令行处理。 

（8）根据类型来设置属性ro.secure的值

```shell

## user/userdebug ##

user_variant := $(filter user userdebug,$(TARGET_BUILD_VARIANT))
enable_target_debugging := true
tags_to_install :=
ifneq (,$(user_variant))
  # Target is secure in user builds.
  ADDITIONAL_DEFAULT_PROPERTIES += ro.secure=1

  ifeq ($(user_variant),userdebug)
    # Pick up some extra useful tools
    tags_to_install += debug

    # Enable Dalvik lock contention logging for userdebug builds.
    ADDITIONAL_BUILD_PROPERTIES += dalvik.vm.lockprof.threshold=500
  else
    # Disable debugging in plain user builds.
    enable_target_debugging :=
  endif

  # Turn on Dalvik preoptimization for libdvm.so user builds, but only if not
  # explicitly disabled and the build is running on Linux (since host
  # Dalvik isn't built for non-Linux hosts).
  ifeq (,$(WITH_DEXPREOPT))
    ifeq ($(DALVIK_VM_LIB),libdvm.so)
      ifeq ($(user_variant),user)
        ifeq ($(HOST_OS),linux)
          WITH_DEXPREOPT := true
        endif
      endif
    endif
  endif

  # Disallow mock locations by default for user builds
  ADDITIONAL_DEFAULT_PROPERTIES += ro.allow.mock.location=0

else # !user_variant
  # Turn on checkjni for non-user builds.
  ADDITIONAL_BUILD_PROPERTIES += ro.kernel.android.checkjni=1
  # Set device insecure for non-user builds.
  ADDITIONAL_DEFAULT_PROPERTIES += ro.secure=0
  # Allow mock locations by default for non user builds
  ADDITIONAL_DEFAULT_PROPERTIES += ro.allow.mock.location=1
endif # !user_variant

ifeq (true,$(strip $(enable_target_debugging)))
  # Target is more debuggable and adbd is on by default
  ADDITIONAL_DEFAULT_PROPERTIES += ro.debuggable=1
  # Include the debugging/testing OTA keys in this build.
  INCLUDE_TEST_OTA_KEYS := true
else # !enable_target_debugging
  # Target is less debuggable and adbd is off by default
  ADDITIONAL_DEFAULT_PROPERTIES += ro.debuggable=0
endif # !enable_target_debugging

## eng ##

ifeq ($(TARGET_BUILD_VARIANT),eng)
tags_to_install := debug eng
ifneq ($(filter ro.setupwizard.mode=ENABLED, $(call collapse-pairs, $(ADDITIONAL_BUILD_PROPERTIES))),)
  # Don't require the setup wizard on eng builds
  ADDITIONAL_BUILD_PROPERTIES := $(filter-out ro.setupwizard.mode=%,\
          $(call collapse-pairs, $(ADDITIONAL_BUILD_PROPERTIES))) \
          ro.setupwizard.mode=OPTIONAL
endif
ifndef is_sdk_build
  # Don't even verify the image on eng builds to speed startup
  ADDITIONAL_BUILD_PROPERTIES += dalvik.vm.image-dex2oat-filter=verify-none
  # Don't compile apps on eng builds to speed startup
  ADDITIONAL_BUILD_PROPERTIES += dalvik.vm.dex2oat-filter=interpret-only
endif
endif

## sdk ##
```

（9） 包含进 post_clean.mk和legacypre_builts.mk脚本。根据legacypre_builts.mk中定义的变量GRANDFATHERED_ALL_PREBUILT 检查是否有不在这个列表中的prebuilt模块。如果有则报错推出。

```shell
# Now with all Android.mks loaded we can do post cleaning steps.
include $(BUILD_SYSTEM)/post_clean.mk

ifeq ($(stash_product_vars),true)
  $(call assert-product-vars, __STASHED)
endif

include $(BUILD_SYSTEM)/legacy_prebuilts.mk
ifneq ($(filter-out $(GRANDFATHERED_ALL_PREBUILT),$(strip $(notdir $(ALL_PREBUILT)))),)
  $(warning *** Some files have been added to ALL_PREBUILT.)
  $(warning *)
  $(warning * ALL_PREBUILT is a deprecated mechanism that)
  $(warning * should not be used for new files.)
  $(warning * As an alternative, use PRODUCT_COPY_FILES in)
  $(warning * the appropriate product definition.)
  $(warning * build/target/product/core.mk is the product)
  $(warning * definition used in all products.)
  $(warning *)
  $(foreach bad_prebuilt,$(filter-out $(GRANDFATHERED_ALL_PREBUILT),$(strip $(notdir $(ALL_PREBUILT)))),$(warning * unexpected $(bad_prebuilt) in ALL_PREBUILT))
  $(warning *)
  $(error ALL_PREBUILT contains unexpected files)
endif

```

设置系统如果没响应，跟踪文件的路径

```shell
# enable vm tracing in files for now to help track
# the cause of ANRs in the content process
ADDITIONAL_BUILD_PROPERTIES += dalvik.vm.stack-trace-file=/data/anr/traces.txt
```

（10）计算那些模块应该在本次编译中引入：

```shell
# -------------------------------------------------------------------
# Fix up CUSTOM_MODULES to refer to installed files rather than
# just bare module names.  Leave unknown modules alone in case
# they're actually full paths to a particular file.
known_custom_modules := $(filter $(ALL_MODULES),$(CUSTOM_MODULES))
unknown_custom_modules := $(filter-out $(ALL_MODULES),$(CUSTOM_MODULES))
CUSTOM_MODULES := \
	$(call module-installed-files,$(known_custom_modules)) \
	$(unknown_custom_modules)

# -------------------------------------------------------------------
# Define dependencies for modules that require other modules.
# This can only happen now, after we've read in all module makefiles.
#
# TODO: deal with the fact that a bare module name isn't
# unambiguous enough.  Maybe declare short targets like
# APPS:Quake or HOST:SHARED_LIBRARIES:libutils.
# BUG: the system image won't know to depend on modules that are
# brought in as requirements of other modules.
#
# Resolve the required module name to 32-bit or 64-bit variant.
# Get a list of corresponding 32-bit module names, if one exists.
define get-32-bit-modules
$(strip $(foreach m,$(1),\
  $(if $(ALL_MODULES.$(m)$(TARGET_2ND_ARCH_MODULE_SUFFIX).CLASS),\
    $(m)$(TARGET_2ND_ARCH_MODULE_SUFFIX))))
endef
# Get a list of corresponding 32-bit module names, if one exists;
# otherwise return the original module name
define get-32-bit-modules-if-we-can
$(strip $(foreach m,$(1),\
  $(if $(ALL_MODULES.$(m)$(TARGET_2ND_ARCH_MODULE_SUFFIX).CLASS),\
    $(m)$(TARGET_2ND_ARCH_MODULE_SUFFIX),
    $(m))))
endef

# If a module is built for 32-bit, the required modules must be 32-bit too;
# Otherwise if the module is an exectuable or shared library,
#   the required modules must be 64-bit;
#   otherwise we require both 64-bit and 32-bit variant, if one exists.
$(foreach m,$(ALL_MODULES),\
  $(eval r := $(ALL_MODULES.$(m).REQUIRED))\
  $(if $(r),\
    $(if $(ALL_MODULES.$(m).FOR_2ND_ARCH),\
      $(eval r_r := $(call get-32-bit-modules-if-we-can,$(r))),\
      $(if $(filter EXECUTABLES SHARED_LIBRARIES,$(ALL_MODULES.$(m).CLASS)),\
        $(eval r_r := $(r)),\
        $(eval r_r := $(r) $(call get-32-bit-modules,$(r)))\
       )\
     )\
     $(eval ALL_MODULES.$(m).REQUIRED := $(strip $(r_r)))\
  )\
)
r_r :=

define add-required-deps
$(1): | $(2)
endef

$(foreach m,$(ALL_MODULES), \
  $(eval r := $(ALL_MODULES.$(m).REQUIRED)) \
  $(if $(r), \
    $(eval r := $(call module-installed-files,$(r))) \
    $(eval t_m := $(filter $(TARGET_OUT_ROOT)/%, $(ALL_MODULES.$(m).INSTALLED))) \
    $(eval h_m := $(filter $(HOST_OUT_ROOT)/%, $(ALL_MODULES.$(m).INSTALLED))) \
    $(eval t_r := $(filter $(TARGET_OUT_ROOT)/%, $(r))) \
    $(eval h_r := $(filter $(HOST_OUT_ROOT)/%, $(r))) \
    $(eval t_m := $(filter-out $(t_r), $(t_m))) \
    $(eval h_m := $(filter-out $(h_r), $(h_m))) \
    $(if $(t_m), $(eval $(call add-required-deps, $(t_m),$(t_r)))) \
    $(if $(h_m), $(eval $(call add-required-deps, $(h_m),$(h_r)))) \
   ) \
 )

t_m :=
h_m :=
t_r :=
h_r :=

```

（11） 包含makefile文件，至此，所有编译文件都包含进来了。

```shell
# build/core/Makefile contains extra stuff that we don't want to pollute this
# top-level makefile with.  It expects that ALL_DEFAULT_INSTALLED_MODULES
# contains everything that's built during the current make, but it also further
# extends ALL_DEFAULT_INSTALLED_MODULES.
ALL_DEFAULT_INSTALLED_MODULES := $(modules_to_install)
include $(BUILD_SYSTEM)/Makefile
modules_to_install := $(sort $(ALL_DEFAULT_INSTALLED_MODULES))
ALL_DEFAULT_INSTALLED_MODULES :=
```

（12）定义系统的编译目标，鉴于内容太多，就不详细列举了，

```shell
# -------------------------------------------------------------------
# This is used to to get the ordering right, you can also use these,
# but they're considered undocumented, so don't complain if their
# behavior changes.
.PHONY: prebuilt
prebuilt: $(ALL_PREBUILT)

# An internal target that depends on all copied headers
# (see copy_headers.make).  Other targets that need the
# headers to be copied first can depend on this target.
.PHONY: all_copied_headers
all_copied_headers: ;

$(ALL_C_CPP_ETC_OBJECTS): | all_copied_headers

# All the droid stuff, in directories
.PHONY: files
files: prebuilt \
        $(modules_to_install) \
        $(INSTALLED_ANDROID_INFO_TXT_TARGET)

# -------------------------------------------------------------------

.PHONY: checkbuild
checkbuild: $(modules_to_check)
ifeq (true,$(ANDROID_BUILD_EVERYTHING_BY_DEFAULT)$(filter $(MAKECMDGOALS),checkbuild))
droid: checkbuild
else
# ANDROID_BUILD_EVERYTHING_BY_DEFAULT not set, or checkbuild is one of the cmd goals.
checkbuild: droid
endif

.PHONY: ramdisk
ramdisk: $(INSTALLED_RAMDISK_TARGET)

.PHONY: factory_ramdisk
factory_ramdisk: $(INSTALLED_FACTORY_RAMDISK_TARGET)

.PHONY: factory_bundle
factory_bundle: $(INSTALLED_FACTORY_BUNDLE_TARGET)

.PHONY: systemtarball
systemtarball: $(INSTALLED_SYSTEMTARBALL_TARGET)

.PHONY: boottarball
boottarball: $(INSTALLED_BOOTTARBALL_TARGET)

.PHONY: userdataimage
userdataimage: $(INSTALLED_USERDATAIMAGE_TARGET)

ifneq (,$(filter userdataimage, $(MAKECMDGOALS)))
$(call dist-for-goals, userdataimage, $(BUILT_USERDATAIMAGE_TARGET))
endif

.PHONY: userdatatarball
userdatatarball: $(INSTALLED_USERDATATARBALL_TARGET)

.PHONY: cacheimage
cacheimage: $(INSTALLED_CACHEIMAGE_TARGET)

.PHONY: vendorimage
vendorimage: $(INSTALLED_VENDORIMAGE_TARGET)

.PHONY: bootimage
bootimage: $(INSTALLED_BOOTIMAGE_TARGET)

# phony target that include any targets in $(ALL_MODULES)
.PHONY: all_modules
ifndef BUILD_MODULES_IN_PATHS
all_modules: $(ALL_MODULES)
else
# BUILD_MODULES_IN_PATHS is a list of paths relative to the top of the tree
module_path_patterns := $(foreach p, $(BUILD_MODULES_IN_PATHS),\
    $(if $(filter %/,$(p)),$(p)%,$(p)/%))
my_all_modules := $(sort $(foreach m, $(ALL_MODULES),$(if $(filter\
    $(module_path_patterns), $(addsuffix /,$(ALL_MODULES.$(m).PATH))),$(m))))
all_modules: $(my_all_modules)
endif


# Build files and then package it into the rom formats
.PHONY: droidcore
droidcore: files \
	systemimage \
	$(INSTALLED_BOOTIMAGE_TARGET) \
	$(INSTALLED_RECOVERYIMAGE_TARGET) \
	$(INSTALLED_USERDATAIMAGE_TARGET) \
	$(INSTALLED_CACHEIMAGE_TARGET) \
	$(INSTALLED_VENDORIMAGE_TARGET) \
	$(INSTALLED_FILES_FILE)

# dist_files only for putting your library into the dist directory with a full build.
.PHONY: dist_files

ifneq ($(TARGET_BUILD_APPS),)
  # If this build is just for apps, only build apps and not the full system by default.

  unbundled_build_modules :=
  ifneq ($(filter all,$(TARGET_BUILD_APPS)),)
    # If they used the magic goal "all" then build all apps in the source tree.
    unbundled_build_modules := $(foreach m,$(sort $(ALL_MODULES)),$(if $(filter APPS,$(ALL_MODULES.$(m).CLASS)),$(m)))
  else
    unbundled_build_modules := $(TARGET_BUILD_APPS)
  endif

  # Dist the installed files if they exist.
  apps_only_installed_files := $(foreach m,$(unbundled_build_modules),$(ALL_MODULES.$(m).INSTALLED))
  $(call dist-for-goals,apps_only, $(apps_only_installed_files))
  # For uninstallable modules such as static Java library, we have to dist the built file,
  # as <module_name>.<suffix>
  apps_only_dist_built_files := $(foreach m,$(unbundled_build_modules),$(if $(ALL_MODULES.$(m).INSTALLED),,\
      $(if $(ALL_MODULES.$(m).BUILT),$(ALL_MODULES.$(m).BUILT):$(m)$(suffix $(ALL_MODULES.$(m).BUILT)))))
  $(call dist-for-goals,apps_only, $(apps_only_dist_built_files))

  ifeq ($(EMMA_INSTRUMENT),true)
    $(EMMA_META_ZIP) : $(apps_only_installed_files)

    $(call dist-for-goals,apps_only, $(EMMA_META_ZIP))
  endif

  $(PROGUARD_DICT_ZIP) : $(apps_only_installed_files)
  $(call dist-for-goals,apps_only, $(PROGUARD_DICT_ZIP))

  $(SYMBOLS_ZIP) : $(apps_only_installed_files)
  $(call dist-for-goals,apps_only, $(SYMBOLS_ZIP))

.PHONY: apps_only
apps_only: $(unbundled_build_modules)

droid: apps_only

# Combine the NOTICE files for a apps_only build
$(eval $(call combine-notice-files, \
    $(target_notice_file_txt), \
    $(target_notice_file_html), \
    "Notices for files for apps:", \
    $(TARGET_OUT_NOTICE_FILES), \
    $(apps_only_installed_files)))


else # TARGET_BUILD_APPS
  $(call dist-for-goals, droidcore, \
    $(INTERNAL_UPDATE_PACKAGE_TARGET) \
    $(INTERNAL_OTA_PACKAGE_TARGET) \
    $(SYMBOLS_ZIP) \
    $(INSTALLED_FILES_FILE) \
    $(INSTALLED_BUILD_PROP_TARGET) \
    $(BUILT_TARGET_FILES_PACKAGE) \
    $(INSTALLED_ANDROID_INFO_TXT_TARGET) \
    $(INSTALLED_RAMDISK_TARGET) \
    $(INSTALLED_FACTORY_RAMDISK_TARGET) \
    $(INSTALLED_FACTORY_BUNDLE_TARGET) \
   )

  # Put a copy of the radio/bootloader files in the dist dir.
  $(foreach f,$(INSTALLED_RADIOIMAGE_TARGET), \
    $(call dist-for-goals, droidcore, $(f)))

  ifneq ($(ANDROID_BUILD_EMBEDDED),true)
  ifneq ($(TARGET_BUILD_PDK),true)
    $(call dist-for-goals, droidcore, \
      $(APPS_ZIP) \
      $(INTERNAL_EMULATOR_PACKAGE_TARGET) \
      $(PACKAGE_STATS_FILE) \
    )
  endif
  endif

  ifeq ($(EMMA_INSTRUMENT),true)
    $(EMMA_META_ZIP) : $(INSTALLED_SYSTEMIMAGE)

    $(call dist-for-goals, dist_files, $(EMMA_META_ZIP))
  endif

# Building a full system-- the default is to build droidcore
droid: droidcore dist_files

endif # TARGET_BUILD_APPS

.PHONY: docs
docs: $(ALL_DOCS)

.PHONY: sdk
ALL_SDK_TARGETS := $(INTERNAL_SDK_TARGET)
sdk: $(ALL_SDK_TARGETS)
$(call dist-for-goals,sdk win_sdk, \
    $(ALL_SDK_TARGETS) \
    $(SYMBOLS_ZIP) \
    $(INSTALLED_BUILD_PROP_TARGET) \
)

# umbrella targets to assit engineers in verifying builds
.PHONY: java native target host java-host java-target native-host native-target \
        java-host-tests java-target-tests native-host-tests native-target-tests \
        java-tests native-tests host-tests target-tests tests
# some synonyms
.PHONY: host-java target-java host-native target-native \
        target-java-tests target-native-tests
host-java : java-host
target-java : java-target
host-native : native-host
target-native : native-target
target-java-tests : java-target-tests
target-native-tests : native-target-tests
tests : host-tests target-tests

# To catch more build breakage, check build tests modules in eng and userdebug builds.
ifneq ($(TARGET_BUILD_PDK),true)
ifneq ($(filter eng userdebug,$(TARGET_BUILD_VARIANT)),)
droidcore : target-tests host-tests
endif
endif

.PHONY: lintall

ifneq (,$(filter samplecode, $(MAKECMDGOALS)))
.PHONY: samplecode
sample_MODULES := $(sort $(call get-tagged-modules,samples))
sample_APKS_DEST_PATH := $(TARGET_COMMON_OUT_ROOT)/samples
sample_APKS_COLLECTION := \
        $(foreach module,$(sample_MODULES),$(sample_APKS_DEST_PATH)/$(notdir $(module)))
$(foreach module,$(sample_MODULES),$(eval $(call \
        copy-one-file,$(module),$(sample_APKS_DEST_PATH)/$(notdir $(module)))))
sample_ADDITIONAL_INSTALLED := \
        $(filter-out $(modules_to_install) $(modules_to_check) $(ALL_PREBUILT),$(sample_MODULES))
samplecode: $(sample_APKS_COLLECTION)
	@echo "Collect sample code apks: $^"
	# remove apks that are not intended to be installed.
	rm -f $(sample_ADDITIONAL_INSTALLED)
endif  # samplecode in $(MAKECMDGOALS)

.PHONY: findbugs
findbugs: $(INTERNAL_FINDBUGS_HTML_TARGET) $(INTERNAL_FINDBUGS_XML_TARGET)

.PHONY: clean
clean:
	@rm -rf $(OUT_DIR)/*
	@echo "Entire build directory removed."

.PHONY: clobber
clobber: clean

# The rules for dataclean and installclean are defined in cleanbuild.mk.

#xxx scrape this from ALL_MODULE_NAME_TAGS
.PHONY: modules
modules:
	@echo "Available sub-modules:"
	@echo "$(call module-names-for-tag-list,$(ALL_MODULE_TAGS))" | \
	      tr -s ' ' '\n' | sort -u | $(COLUMN)

.PHONY: showcommands
showcommands:
	@echo >/dev/null

.PHONY: nothing
nothing:
	@echo Successfully read the makefiles.
```

##### 2.15 Build 系统的编译目标介绍

 Android Build系统的缺省编译目标是droid。droid目标会依赖其他目标，所有这些目标共同组成了最终的产品。

下面是droid的目标定义

```shell
# This is used to to get the ordering right, you can also use these,
# but they're considered undocumented, so don't complain if their
# behavior changes.
.PHONY: prebuilt
prebuilt: $(ALL_PREBUILT)

# Build files and then package it into the rom formats
.PHONY: droidcore
droidcore: files \
	systemimage \
	$(INSTALLED_BOOTIMAGE_TARGET) \
	$(INSTALLED_RECOVERYIMAGE_TARGET) \
	$(INSTALLED_USERDATAIMAGE_TARGET) \
	$(INSTALLED_CACHEIMAGE_TARGET) \
	$(INSTALLED_VENDORIMAGE_TARGET) \
	$(INSTALLED_FILES_FILE)
# dist_files only for putting your library into the dist directory with a full build.
.PHONY: dist_files
	
# Building a full system-- the default is to build droidcore
droid: droidcore dist_files
# All the droid stuff, in directories
.PHONY: files
files: prebuilt \
        $(modules_to_install) \
        $(INSTALLED_ANDROID_INFO_TXT_TARGET)
	
```

![1533712945811](/img/1533712945811.png)

![1533713394294](/img/1533713394294.png)

##### 2.1.6 分析config.mk文件

> 接下来，在看看config.mk文件，这个文件相当于Build系统的配置文件。

(1) 定义表示文档、头文件、系统库的源码目录等变量，方便其他编译脚本使用

```shell
# Standard source directories.
SRC_DOCS:= $(TOPDIR)docs
# TODO: Enforce some kind of layering; only add include paths
#       when a module links against a particular library.
# TODO: See if we can remove most of these from the global list.
SRC_HEADERS := \
        $(TOPDIR)system/core/include \
        $(TOPDIR)hardware/libhardware/include \
        $(TOPDIR)hardware/libhardware_legacy/include \
        $(TOPDIR)hardware/ril/include \
        $(TOPDIR)libnativehelper/include \
        $(TOPDIR)frameworks/native/include \
        $(TOPDIR)frameworks/native/opengl/include \
        $(TOPDIR)frameworks/av/include \
        $(TOPDIR)frameworks/base/include
SRC_HOST_HEADERS:=$(TOPDIR)tools/include
SRC_LIBRARIES:= $(TOPDIR)libs
SRC_SERVERS:= $(TOPDIR)servers
SRC_TARGET_DIR := $(TOPDIR)build/target
SRC_API_DIR := $(TOPDIR)prebuilts/sdk/api
SRC_SYSTEM_API_DIR := $(TOPDIR)prebuilts/sdk/system-api

# Some specific paths to tools
SRC_DROIDDOC_DIR := $(TOPDIR)build/tools/droiddoc

```

(2) 包含pathmap.mk文件

```shell
# Various mappings to avoid hard-coding paths all over the place
include $(BUILD_SYSTEM)/pathmap.mk
```

(3) 定义模块编译变量名，这里的模块是指编译出的apk文件，静态java库，共享Java库等。这些变量会用在这些模块的编译脚本中，各个模块编译脚本的结尾都会include下面一个变量名，这将包含进某个系统的编译脚本：

```shell
# Build system internal files
# ###############################################################

BUILD_COMBOS:= $(BUILD_SYSTEM)/combo

CLEAR_VARS:= $(BUILD_SYSTEM)/clear_vars.mk
BUILD_HOST_STATIC_LIBRARY:= $(BUILD_SYSTEM)/host_static_library.mk
BUILD_HOST_SHARED_LIBRARY:= $(BUILD_SYSTEM)/host_shared_library.mk
BUILD_STATIC_LIBRARY:= $(BUILD_SYSTEM)/static_library.mk
BUILD_RAW_STATIC_LIBRARY := $(BUILD_SYSTEM)/raw_static_library.mk
BUILD_SHARED_LIBRARY:= $(BUILD_SYSTEM)/shared_library.mk
BUILD_EXECUTABLE:= $(BUILD_SYSTEM)/executable.mk
BUILD_RAW_EXECUTABLE:= $(BUILD_SYSTEM)/raw_executable.mk
BUILD_HOST_EXECUTABLE:= $(BUILD_SYSTEM)/host_executable.mk
BUILD_PACKAGE:= $(BUILD_SYSTEM)/package.mk
BUILD_PHONY_PACKAGE:= $(BUILD_SYSTEM)/phony_package.mk
BUILD_HOST_PREBUILT:= $(BUILD_SYSTEM)/host_prebuilt.mk
BUILD_PREBUILT:= $(BUILD_SYSTEM)/prebuilt.mk
BUILD_MULTI_PREBUILT:= $(BUILD_SYSTEM)/multi_prebuilt.mk
BUILD_JAVA_LIBRARY:= $(BUILD_SYSTEM)/java_library.mk
BUILD_STATIC_JAVA_LIBRARY:= $(BUILD_SYSTEM)/static_java_library.mk
BUILD_HOST_JAVA_LIBRARY:= $(BUILD_SYSTEM)/host_java_library.mk
BUILD_DROIDDOC:= $(BUILD_SYSTEM)/droiddoc.mk
BUILD_COPY_HEADERS := $(BUILD_SYSTEM)/copy_headers.mk
BUILD_NATIVE_TEST := $(BUILD_SYSTEM)/native_test.mk
BUILD_HOST_NATIVE_TEST := $(BUILD_SYSTEM)/host_native_test.mk

BUILD_SHARED_TEST_LIBRARY := $(BUILD_SYSTEM)/shared_test_lib.mk
BUILD_HOST_SHARED_TEST_LIBRARY := $(BUILD_SYSTEM)/host_shared_test_lib.mk
BUILD_STATIC_TEST_LIBRARY := $(BUILD_SYSTEM)/static_test_lib.mk
BUILD_HOST_STATIC_TEST_LIBRARY := $(BUILD_SYSTEM)/host_static_test_lib.mk

BUILD_NOTICE_FILE := $(BUILD_SYSTEM)/notice_files.mk
BUILD_HOST_DALVIK_JAVA_LIBRARY := $(BUILD_SYSTEM)/host_dalvik_java_library.mk
BUILD_HOST_DALVIK_STATIC_JAVA_LIBRARY := $(BUILD_SYSTEM)/host_dalvik_static_java_library.mk


```

在某些5.0系统版本中，还会配置dalvik虚拟机的动态库

![1533719327526](/img/1533719327526.png)

(4) 定义c/c++代码编译时的参数以及系统常用包的后缀名：

```shell

# ###############################################################
# Set common values
# ###############################################################

# These can be changed to modify both host and device modules.
COMMON_GLOBAL_CFLAGS:= -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith
COMMON_RELEASE_CFLAGS:= -DNDEBUG -UDEBUG

COMMON_GLOBAL_CPPFLAGS:= $(COMMON_GLOBAL_CFLAGS) -Wsign-promo
COMMON_RELEASE_CPPFLAGS:= $(COMMON_RELEASE_CFLAGS)

# Set the extensions used for various packages
COMMON_PACKAGE_SUFFIX := .zip
COMMON_JAVA_PACKAGE_SUFFIX := .jar
COMMON_ANDROID_PACKAGE_SUFFIX := .apk

# list of flags to turn specific warnings in to errors
TARGET_ERROR_FLAGS := -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point

# TODO: do symbol compression
TARGET_COMPRESS_MODULE_SYMBOLS := false


# Only use ANDROID_BUILD_SHELL to wrap around bash.
# DO NOT use other shells such as zsh.
ifdef ANDROID_BUILD_SHELL
SHELL := $(ANDROID_BUILD_SHELL)
else
# Use bash, not whatever shell somebody has installed as /bin/sh
# This is repeated from main.mk, since envsetup.sh runs this file
# directly.
SHELL := /bin/bash
endif
```

(5) 如果在源码根目录下有buildspec.mk文件，包含进来：

```shell
ifndef ANDROID_BUILDSPEC
ANDROID_BUILDSPEC := $(TOPDIR)buildspec.mk
endif
-include $(ANDROID_BUILDSPEC)

```

(6) 包含envsetup.sh文件

```shell
include $(BUILD_SYSTEM)/envsetup.mk
```

(7) 包含select.mk文件。注意这里一共包含了4次，但是每次包含前会对变量



```shell

# $(1): os/arch
define select-android-config-h
build/core/combo/include/arch/$(1)/AndroidConfig.h
endef

combo_target := HOST_
combo_2nd_arch_prefix :=
include $(BUILD_SYSTEM)/combo/select.mk

# Load the 2nd host arch if it's needed.
ifdef HOST_2ND_ARCH
combo_target := HOST_
combo_2nd_arch_prefix := $(HOST_2ND_ARCH_VAR_PREFIX)
include $(BUILD_SYSTEM)/combo/select.mk
endif

# on windows, the tools have .exe at the end, and we depend on the
# host config stuff being done first

combo_target := TARGET_
combo_2nd_arch_prefix :=
include $(BUILD_SYSTEM)/combo/select.mk

# Load the 2nd target arch if it's needed.
ifdef TARGET_2ND_ARCH
combo_target := TARGET_
combo_2nd_arch_prefix := $(TARGET_2ND_ARCH_VAR_PREFIX)
include $(BUILD_SYSTEM)/combo/select.mk
endif

ifdef TARGET_PREFER_32_BIT
TARGET_PREFER_32_BIT_APPS := true
TARGET_PREFER_32_BIT_EXECUTABLES := true
endif

ifeq (,$(TARGET_SUPPORTS_32_BIT_APPS)$(TARGET_SUPPORTS_64_BIT_APPS))
  TARGET_SUPPORTS_32_BIT_APPS := true
endif

```

(8) 包含javac.mk文件，这个文件定义了Java编译工具的路劲

```shell
# Pick a Java compiler.
include $(BUILD_SYSTEM)/combo/javac.mk
```

(9) 定义Build系统使用的一些工具的路径

```shell
# ---------------------------------------------------------------
# Generic tools.

LEX := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/flex/flex-2.5.39
# The default PKGDATADIR built in the prebuilt bison is a relative path
# external/bison/data.
# To run bison from elsewhere you need to set up enviromental variable
# BISON_PKGDATADIR.
BISON_PKGDATADIR := $(PWD)/external/bison/data
BISON := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/bison/bison
YACC := $(BISON) -d

YASM := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/yasm/yasm

DOXYGEN:= doxygen
AAPT := $(HOST_OUT_EXECUTABLES)/aapt$(HOST_EXECUTABLE_SUFFIX)
AIDL := $(HOST_OUT_EXECUTABLES)/aidl$(HOST_EXECUTABLE_SUFFIX)
PROTOC := $(HOST_OUT_EXECUTABLES)/aprotoc$(HOST_EXECUTABLE_SUFFIX)
SIGNAPK_JAR := $(HOST_OUT_JAVA_LIBRARIES)/signapk$(COMMON_JAVA_PACKAGE_SUFFIX)
MKBOOTFS := $(HOST_OUT_EXECUTABLES)/mkbootfs$(HOST_EXECUTABLE_SUFFIX)
MINIGZIP := $(HOST_OUT_EXECUTABLES)/minigzip$(HOST_EXECUTABLE_SUFFIX)
ifeq (,$(strip $(BOARD_CUSTOM_MKBOOTIMG)))
MKBOOTIMG := $(HOST_OUT_EXECUTABLES)/mkbootimg$(HOST_EXECUTABLE_SUFFIX)
else
MKBOOTIMG := $(BOARD_CUSTOM_MKBOOTIMG)
endif
MKYAFFS2 := $(HOST_OUT_EXECUTABLES)/mkyaffs2image$(HOST_EXECUTABLE_SUFFIX)
APICHECK := $(HOST_OUT_EXECUTABLES)/apicheck$(HOST_EXECUTABLE_SUFFIX)
FS_GET_STATS := $(HOST_OUT_EXECUTABLES)/fs_get_stats$(HOST_EXECUTABLE_SUFFIX)
MKEXT2IMG := $(HOST_OUT_EXECUTABLES)/genext2fs$(HOST_EXECUTABLE_SUFFIX)
MAKE_EXT4FS := $(HOST_OUT_EXECUTABLES)/make_ext4fs$(HOST_EXECUTABLE_SUFFIX)
MKEXTUSERIMG := $(HOST_OUT_EXECUTABLES)/mkuserimg.sh
MAKE_F2FS := $(HOST_OUT_EXECUTABLES)/make_f2fs$(HOST_EXECUTABLE_SUFFIX)
MKF2FSUSERIMG := $(HOST_OUT_EXECUTABLES)/mkf2fsuserimg.sh
MKEXT2BOOTIMG := external/genext2fs/mkbootimg_ext2.sh
SIMG2IMG := $(HOST_OUT_EXECUTABLES)/simg2img$(HOST_EXECUTABLE_SUFFIX)
E2FSCK := $(HOST_OUT_EXECUTABLES)/e2fsck$(HOST_EXECUTABLE_SUFFIX)
MKTARBALL := build/tools/mktarball.sh
TUNE2FS := $(HOST_OUT_EXECUTABLES)/tune2fs$(HOST_EXECUTABLE_SUFFIX)
E2FSCK := $(HOST_OUT_EXECUTABLES)/e2fsck$(HOST_EXECUTABLE_SUFFIX)
JARJAR := $(HOST_OUT_JAVA_LIBRARIES)/jarjar.jar
PROGUARD := external/proguard/bin/proguard.sh
JAVATAGS := build/tools/java-event-log-tags.py
LLVM_RS_CC := $(HOST_OUT_EXECUTABLES)/llvm-rs-cc$(HOST_EXECUTABLE_SUFFIX)
BCC_COMPAT := $(HOST_OUT_EXECUTABLES)/bcc_compat$(HOST_EXECUTABLE_SUFFIX)
LINT := prebuilts/sdk/tools/lint
RMTYPEDEFS := $(HOST_OUT_EXECUTABLES)/rmtypedefs
APPEND2SIMG := $(HOST_OUT_EXECUTABLES)/append2simg
VERITY_SIGNER := $(HOST_OUT_EXECUTABLES)/verity_signer
BUILD_VERITY_TREE := $(HOST_OUT_EXECUTABLES)/build_verity_tree
BOOT_SIGNER := $(HOST_OUT_EXECUTABLES)/boot_signer

# ACP is always for the build OS, not for the host OS
ACP := $(BUILD_OUT_EXECUTABLES)/acp$(BUILD_EXECUTABLE_SUFFIX)

# dx is java behind a shell script; no .exe necessary.
DX := $(HOST_OUT_EXECUTABLES)/dx
ZIPALIGN := $(HOST_OUT_EXECUTABLES)/zipalign$(HOST_EXECUTABLE_SUFFIX)
FINDBUGS_DIR := external/owasp/sanitizer/tools/findbugs/bin
FINDBUGS := $(FINDBUGS_DIR)/findbugs
EMMA_JAR := external/emma/lib/emma$(COMMON_JAVA_PACKAGE_SUFFIX)

# Tool to merge AndroidManifest.xmls
ANDROID_MANIFEST_MERGER := java -classpath prebuilts/devtools/tools/lib/manifest-merger.jar com.android.manifmerger.Main merge

YACC_HEADER_SUFFIX:= .hpp

# Don't use column under Windows, cygwin or not
ifeq ($(HOST_OS),windows)
COLUMN:= cat
else
COLUMN:= column
endif

ifeq ($(HOST_OS),darwin)
ifeq ($(LEGACY_USE_JAVA6),)
HOST_JDK_TOOLS_JAR:= $(shell $(BUILD_SYSTEM)/find-jdk-tools-jar.sh)
else
# Deliberately set to blank for Java 6 installations on MacOS. These
# versions allegedly use a non-standard directory structure.
HOST_JDK_TOOLS_JAR :=
endif
else
HOST_JDK_TOOLS_JAR:= $(shell $(BUILD_SYSTEM)/find-jdk-tools-jar.sh)
endif

ifneq ($(HOST_JDK_TOOLS_JAR),)
ifeq ($(wildcard $(HOST_JDK_TOOLS_JAR)),)
$(error Error: could not find jdk tools.jar, please check if your JDK was installed correctly)
endif
endif

# Is the host JDK 64-bit version?
HOST_JDK_IS_64BIT_VERSION :=
ifneq ($(filter 64-Bit, $(shell java -version 2>&1)),)
HOST_JDK_IS_64BIT_VERSION := true
endif

# It's called md5 on Mac OS and md5sum on Linux
ifeq ($(HOST_OS),darwin)
MD5SUM:=md5 -q
else
MD5SUM:=md5sum
endif

```

> lex和yacc可以帮助你编写程序转换结构化输入。既包括从输入文件中寻找模式的简单文本搜索程序，也包括将源程序变换为最佳的目标代码的C编译程序等。 

(10) 定义host平台和target平台各自编译，链接c/c++使用的参数。下面是第一个，当然还有其他候选的一些默认设置：

```shell

# ###############################################################
# Set up final options.
# ###############################################################

HOST_GLOBAL_CFLAGS += $(COMMON_GLOBAL_CFLAGS)
HOST_RELEASE_CFLAGS += $(COMMON_RELEASE_CFLAGS)

HOST_GLOBAL_CPPFLAGS += $(COMMON_GLOBAL_CPPFLAGS)
HOST_RELEASE_CPPFLAGS += $(COMMON_RELEASE_CPPFLAGS)

TARGET_GLOBAL_CFLAGS += $(COMMON_GLOBAL_CFLAGS)
TARGET_RELEASE_CFLAGS += $(COMMON_RELEASE_CFLAGS)

TARGET_GLOBAL_CPPFLAGS += $(COMMON_GLOBAL_CPPFLAGS)
TARGET_RELEASE_CPPFLAGS += $(COMMON_RELEASE_CPPFLAGS)

HOST_GLOBAL_LD_DIRS += -L$(HOST_OUT_INTERMEDIATE_LIBRARIES)
TARGET_GLOBAL_LD_DIRS += -L$(TARGET_OUT_INTERMEDIATE_LIBRARIES)

HOST_PROJECT_INCLUDES:= $(SRC_HEADERS) $(SRC_HOST_HEADERS) $(HOST_OUT_HEADERS)
TARGET_PROJECT_INCLUDES:= $(SRC_HEADERS) $(TARGET_OUT_HEADERS) \
                $(TARGET_DEVICE_KERNEL_HEADERS) $(TARGET_BOARD_KERNEL_HEADERS) \
                $(TARGET_PRODUCT_KERNEL_HEADERS)

# Many host compilers don't support these flags, so we have to make
# sure to only specify them for the target compilers checked in to
# the source tree.
TARGET_GLOBAL_CFLAGS += $(TARGET_ERROR_FLAGS)
TARGET_GLOBAL_CPPFLAGS += $(TARGET_ERROR_FLAGS)

HOST_GLOBAL_CFLAGS += $(HOST_RELEASE_CFLAGS)
HOST_GLOBAL_CPPFLAGS += $(HOST_RELEASE_CPPFLAGS)

TARGET_GLOBAL_CFLAGS += $(TARGET_RELEASE_CFLAGS)
TARGET_GLOBAL_CPPFLAGS += $(TARGET_RELEASE_CPPFLAGS)

```

(11) 包含clang/config.mk文件。这个文件定义了llvm编译器clang的路径和参数

```shell
# define clang/llvm tools and global flags
include $(BUILD_SYSTEM)/clang/config.mk
```

(12) 定义 Andoid SDK的版本：

```shell

TARGET_AVAILABLE_SDK_VERSIONS := $(call numerically_sort,\
    $(patsubst $(HISTORICAL_SDK_VERSIONS_ROOT)/%/android.jar,%, \
    $(wildcard $(HISTORICAL_SDK_VERSIONS_ROOT)/*/android.jar)))

# We don't have prebuilt system_current SDK yet.
TARGET_AVAILABLE_SDK_VERSIONS := $(TARGET_AVAILABLE_SDK_VERSIONS)
```

(13) 包含dumpvar.mk文件，打印出本次编译产品的配置信息：

```shell
include $(BUILD_SYSTEM)/dumpvar.mk
```

##### 2.1.7分析product_config.mk文件

product_config.mk文件分段如下。

（1） 编译Android时可以实用lunch命令制定所需要编译的设备，但这不是编译系统唯一的方法。我们可以直接在make命令之后通过参数来指定需要编译的产品。因此，这里首先解析make命令的参数$(MAKECMDGOALS)，格式是PRODUCT-<prodname>-<goal>,相当于设置了变量TARGET_PRODUCT为<prodname>、变量TARGET_BUILD_VARIANT设置为<goal>

```shell
# ---------------------------------------------------------------
# Provide "PRODUCT-<prodname>-<goal>" targets, which lets you build
# a particular configuration without needing to set up the environment.
#
product_goals := $(strip $(filter PRODUCT-%,$(MAKECMDGOALS)))
ifdef product_goals
  # Scrape the product and build names out of the goal,
  # which should be of the form PRODUCT-<productname>-<buildname>.
  #
  ifneq ($(words $(product_goals)),1)
    $(error Only one PRODUCT-* goal may be specified; saw "$(product_goals)")
  endif
  goal_name := $(product_goals)
  product_goals := $(patsubst PRODUCT-%,%,$(product_goals))
  product_goals := $(subst -, ,$(product_goals))
  ifneq ($(words $(product_goals)),2)
    $(error Bad PRODUCT-* goal "$(goal_name)")
  endif

  # The product they want
  TARGET_PRODUCT := $(word 1,$(product_goals))

  # The variant they want
  TARGET_BUILD_VARIANT := $(word 2,$(product_goals))

  ifeq ($(TARGET_BUILD_VARIANT),tests)
    $(error "tests" has been deprecated as a build variant. Use it as a build goal instead.)
  endif

  # The build server wants to do make PRODUCT-dream-installclean
  # which really means TARGET_PRODUCT=dream make installclean.
  ifneq ($(filter-out $(INTERNAL_VALID_VARIANTS),$(TARGET_BUILD_VARIANT)),)
    MAKECMDGOALS := $(MAKECMDGOALS) $(TARGET_BUILD_VARIANT)
    TARGET_BUILD_VARIANT := eng
    default_goal_substitution :=
  else
    default_goal_substitution := $(DEFAULT_GOAL)
  endif

  # Replace the PRODUCT-* goal with the build goal that it refers to.
  # Note that this will ensure that it appears in the same relative
  # position, in case it matters.
  #
  # Note that modifying this will not affect the goals that make will
  # attempt to build, but it's important because we inspect this value
  # in certain situations (like for "make sdk").
  #
  MAKECMDGOALS := $(patsubst $(goal_name),$(default_goal_substitution),$(MAKECMDGOALS))

  # Define a rule for the PRODUCT-* goal, and make it depend on the
  # patched-up command-line goals as well as any other goals that we
  # want to force.
  #
.PHONY: $(goal_name)
$(goal_name): $(MAKECMDGOALS)
endif
# else: Use the value set in the environment or buildspec.mk.


```

（2） 如果make命令的参数格式是APP-<appname>，则相当于设置变量TARGET_BUILD_APPS为<appname>。这将导致系统编译某个App模块，而不是某个产品。

```shell
# ---------------------------------------------------------------
# Provide "APP-<appname>" targets, which lets you build
# an unbundled app.
#
unbundled_goals := $(strip $(filter APP-%,$(MAKECMDGOALS)))
ifdef unbundled_goals
  ifneq ($(words $(unbundled_goals)),1)
    $(error Only one APP-* goal may be specified; saw "$(unbundled_goals)"))
  endif
  TARGET_BUILD_APPS := $(strip $(subst -, ,$(patsubst APP-%,%,$(unbundled_goals))))
  ifneq ($(filter $(DEFAULT_GOAL),$(MAKECMDGOALS)),)
    MAKECMDGOALS := $(patsubst $(unbundled_goals),,$(MAKECMDGOALS))
  else
    MAKECMDGOALS := $(patsubst $(unbundled_goals),$(DEFAULT_GOAL),$(MAKECMDGOALS))
  endif

.PHONY: $(unbundled_goals)
$(unbundled_goals): $(MAKECMDGOALS)
endif # unbundled_goals

# Default to building dalvikvm on hosts that support it...
ifeq ($(HOST_OS),linux)
# ... or if the if the option is already set
ifeq ($(WITH_HOST_DALVIK),)
  WITH_HOST_DALVIK := true
endif
endif


```

（3）包含进3个文件：node_fns.mk、product.mk、device.mk：

```shell
# ---------------------------------------------------------------
# Include the product definitions.
# We need to do this to translate TARGET_PRODUCT into its
# underlying TARGET_DEVICE before we start defining any rules.
#
include $(BUILD_SYSTEM)/node_fns.mk
include $(BUILD_SYSTEM)/product.mk
include $(BUILD_SYSTEM)/device.mk


```

（4）执行$(get-all-product-makefiles)函数

```shell
ifneq ($(strip $(TARGET_BUILD_APPS)),)
# An unbundled app build needs only the core product makefiles.
all_product_configs := $(call get-product-makefiles,\
    $(SRC_TARGET_DIR)/product/AndroidProducts.mk)
else
# Read in all of the product definitions specified by the AndroidProducts.mk
# files in the tree.
all_product_configs := $(get-all-product-makefiles)
endif

```

get-all-product-makefiles函数的定义位于文件product.mk中，这个函数会查找vendor和device目录下所有AndroidProducts.mk文件，打开并读取其中PRODUCT_MAKEFILES变量的值，后面会介绍这个变量的作用。

下面是product.mk中的定义

```shell
#
# Returns the list of all AndroidProducts.mk files.
# $(call ) isn't necessary.
#
define _find-android-products-files
$(shell test -d device && find device -maxdepth 6 -name AndroidProducts.mk) \
  $(shell test -d vendor && find vendor -maxdepth 6 -name AndroidProducts.mk) \
  $(SRC_TARGET_DIR)/product/AndroidProducts.mk
endef

#
# Returns the sorted concatenation of PRODUCT_MAKEFILES
# variables set in the given AndroidProducts.mk files.
# $(1): the list of AndroidProducts.mk files.
#
define get-product-makefiles
$(sort \
  $(foreach f,$(1), \
    $(eval PRODUCT_MAKEFILES :=) \
    $(eval LOCAL_DIR := $(patsubst %/,%,$(dir $(f)))) \
    $(eval include $(f)) \
    $(PRODUCT_MAKEFILES) \
   ) \
  $(eval PRODUCT_MAKEFILES :=) \
  $(eval LOCAL_DIR :=) \
 )
endef

#
# Returns the sorted concatenation of all PRODUCT_MAKEFILES
# variables set in all AndroidProducts.mk files.
# $(call ) isn't necessary.
#
define get-all-product-makefiles
$(call get-product-makefiles,$(_find-android-products-files))
endef

```

（5）下面是对current_product_makefile 和all_product_makefiles 两个变量赋值。all_product_makefiles 变量的内容是系统中所产生的配置。current_product_makefile 是当前产品的配置路径。对all_product_makefiles 的赋值是将all_product_configs变量的内容几乎全部复制过来。而对current_product_makefile的赋值则根据$(TARGET_PRODUCT) 的值进行匹配。假如编译师用lunch命令选择了aosp_hammerhead，那么这里current_product_makefile 的值就是 device/lge/hammerhead.mk

```shell

# Find the product config makefile for the current product.
# all_product_configs consists items like:
# <product_name>:<path_to_the_product_makefile>
# or just <path_to_the_product_makefile> in case the product name is the
# same as the base filename of the product config makefile.
current_product_makefile :=
all_product_makefiles :=

$(foreach f, $(all_product_configs),\
    $(eval _cpm_words := $(subst :,$(space),$(f)))\
    $(eval _cpm_word1 := $(word 1,$(_cpm_words)))\
    $(eval _cpm_word2 := $(word 2,$(_cpm_words)))\
    $(if $(_cpm_word2),\
        $(eval all_product_makefiles += $(_cpm_word2))\
        $(if $(filter $(TARGET_PRODUCT),$(_cpm_word1)),\
            $(eval current_product_makefile += $(_cpm_word2)),),\
        $(eval all_product_makefiles += $(f))\
        $(if $(filter $(TARGET_PRODUCT),$(basename $(notdir $(f)))),\
            $(eval current_product_makefile += $(f)),)))
_cpm_words :=
_cpm_word1 :=
_cpm_word2 :=
current_product_makefile := $(strip $(current_product_makefile))
all_product_makefiles := $(strip $(all_product_makefiles))



```

（6）如果make后跟有参数"product-graph" 或者“dump-products”，就会调用

$(call import-products, $(all_product_makefiles))，否则只会执行$(call import-products, $(current_product_makefile))：

```shell

ifneq (,$(filter product-graph dump-products, $(MAKECMDGOALS)))
# Import all product makefiles.
$(call import-products, $(all_product_makefiles))
else
# Import just the current product.
ifndef current_product_makefile
$(error Can not locate config makefile for product "$(TARGET_PRODUCT)")
endif
ifneq (1,$(words $(current_product_makefile)))
$(error Product "$(TARGET_PRODUCT)" ambiguous: matches $(current_product_makefile))
endif
$(call import-products, $(current_product_makefile))
endif  # Import all or just the current product makefile
```

（7）上一步import的结果是产生如PRODUCT.$(TARGET_PRODUCT).xxx的一系列内部变量，然后将它们的值赋予产品相关的变量，例如

```shell
# A list of module names of BOOTCLASSPATH (jar files)
PRODUCT_BOOT_JARS := $(strip $(PRODUCTS.$(INTERNAL_PRODUCT).PRODUCT_BOOT_JARS))
PRODUCT_SYSTEM_SERVER_JARS := $(strip $(PRODUCTS.$(INTERNAL_PRODUCT).PRODUCT_SYSTEM_SERVER_JARS))

# Find the device that this product maps to.
TARGET_DEVICE := $(PRODUCTS.$(INTERNAL_PRODUCT).PRODUCT_DEVICE)

# Figure out which resoure configuration options to use for this
# product.
PRODUCT_LOCALES := $(strip $(PRODUCTS.$(INTERNAL_PRODUCT).PRODUCT_LOCALES))
# TODO: also keep track of things like "port", "land" in product files.


```

理解一个内部变量的办法就是搜索Build系统中所有对该变量赋值和使用的地方，看看系统如何使用这个变量，就能比较精确的掌握这个变量的含义。

##### 2.1.8 android5.0 的64位编译

> Android 5.0开始支持64位编译。当然也允许Android5.0系统在32位的cpu上运行，为了保持兼容性，运行在64位的cpu上时，能同时支持运行32位和64位的应用。因此，理论上就会有4中运行模式。

- 纯32位模式
- 缺省32位模式，同时支持64位模式：需要64位cpu，能与现在的应用最大程序地兼容。
- 缺省64位模式 ，同时支持32位模式：需要64位cpu，能够提供比较好的兼容性，同时最大程度的利用64位的优势。
- 纯64位模式：需要64位cpu，这种模式洗带有32位的动态库的应用将无法应用。

这4中模式在Zygote进程的启动就可以看出。Android5.0一共定义了4中Zygote进程的启动方式，对应这里介绍的4中模式。具体内容后面在分析。

​	对于不带有动态库的apk应用，不用关心系统是32位还是64位。但是杜宇包含有动态库的应用，还是需要考虑将动态库编译成32位和64位的。只要执行lunch命令时选择的产品是64位的，那么编译一个动态库时就会同时产生32位的版本和64位的版本的文件。其中32位版本放在了out/.../system/lib 目录下，64位版本放在了 out/..../system/lib64下。

​      Android5.0中apk优化后的odex文件存放的位置也发生了变化，Android5.0以前apk优化后的odex文件存放在/data/dalvik-cache目录下。Android5.0 后这些文件存放在apk文件所在的目录的"arm",“arm64”目录下。

#### 2.2 Android 的产品配置文件

> 产品配置文件的作用是按照Build 系统的要求，将生成
>
> - 产品的各种image文件所需要的配置信息(如版本号，各种参数等)、
> - 资源(图片、字体、铃声等)、
> - 二进制文件（apk、jar包、so库等）
>
> 有机的组织起来，同时进行剪裁，加入或去掉一些模块。
>
> ​	Android的产品配置文件位于源码的device目录下，但是产品配置文件也可以放在vender目录下。这两个目录从Buld系统的角度看没太大的区别，Build系统中搜索产品配置的关键文件时会同事在这两个目录下进行，但是在实际使用中，往往会让这两个目录配合使用，通常产品配置文件放在device目录下，而vendor目录下则存放一些硬件的HAL库。编译某一款手机的“刷机包”之前，需要将手机上的一些不开源的HAL库（主要是so文件）、驱动等抽取出来，放在vender目录下。

##### 2.2.1 分析hammerhead 的配置文件

```shell
lx@lx-pc:/opt/lollipop-5.1.1_r6-release_3rd/device$ tree -L 2 ./
./
├── asus
│   ├── deb
│   ├── flo
│   ├── flo-kernel
│   ├── fugu
│   ├── fugu-kernel
│   ├── grouper
│   └── tilapia
├── common
│   ├── CleanSpec.mk
│   ├── clear-factory-images-variables.sh
│   ├── generate-blob-lists.sh
│   ├── generate-factory-images-common.sh
│   ├── generate-packages.sh
│   ├── gps
│   └── populate-new-device.sh
├── generic
│   ├── arm64
│   ├── armv7-a-neon
│   ├── common
│   ├── goldfish
│   ├── mini-emulator-arm64
│   ├── mini-emulator-armv7-a-neon
│   ├── mini-emulator-mips
│   ├── mini-emulator-x86
│   ├── mini-emulator-x86_64
│   ├── mips
│   ├── qemu
│   ├── x86
│   └── x86_64
├── google
│   ├── accessory
│   └── atv
├── htc
│   ├── flounder
│   └── flounder-kernel
├── lge
│   ├── hammerhead
│   ├── hammerhead-kernel
│   ├── mako
│   └── mako-kernel
├── moto
│   ├── shamu
│   └── shamu-kernel
├── nexell
│   ├── dckim
│   ├── lepus
│   ├── library
│   ├── library2
│   ├── library_mwsr
│   ├── s5p4418_drone
│   ├── s5p4418_general
│   ├── s5p6818_drone
│   └── s5p6818_general
├── sample
│   ├── Android.mk
│   ├── apps
│   ├── CleanSpec.mk
│   ├── etc
│   ├── frameworks
│   ├── MODULE_LICENSE_APACHE2
│   ├── overlays
│   ├── products
│   ├── README.txt
│   ├── sdk_addon
│   └── skins
└── samsung
    └── manta
```

  通常device目录中有几个子目录。

（1）common：用来存放各个产品通用的配置脚本、文件等。

（2）sample：一个产品配置的例子，写一个新的产品配置时可以使用sample目录下文件作为模板

（3）google：几个简单的模块，用途不详。

（4）generic：存放的用于模拟器的产品，包括x86、arm、mips架构

（5）asus、lge、samsung：分别代表宏碁、LG、三星3家公司。各家公司的产品存放在对应的目录下。

> 如果需要添加新的产品，可以在device的目录下新建一个目录。
>
> hammerhead手机由LG代工的，它的产品配置文件位于lge目录下，具体内容如下。

（1）hammerhead：存放 Google Nexue5的产品的配置文件，也就是下面分析的重点。
（2）hammerhead-kernel：存放的是hammerhead的二进制文件
（3） mako：存放的是Nexus4 的产品配置文件
（4）mako-kernel：存放的nexus4 kernel的image

下面介绍Build系统会包含产品配置中的几个文件，这几个文件和BUild系统关系最为紧密的，也是产品配置的关键文件，整个产品目录的组织就是围绕着这几个文件展开的。

###### 1.vendorsetup.sh

vendorsetup.sh文件会在初始化编译环境时被eventsetup.sh文件包含进去。它主要的作用是调用add_lunch_combo命令来添加产品的名称串。例如harmerhead目录下的vendorsetup.sh文件的内容是：

> add_lunch_combo aosp_hammerhead-userdebug

产品前面通常加上一个"aosp_",这个前缀从编译角度看并无实际意义，它只是产品名称的一部分。AOSP的含义是 android open source project。除了"aosp__" 前缀，还有另一个前缀"full—"。从这里可以看到：即使在同一个产品配置中，也可以非常方便的编译多个不同的版本来。

​	产品的编译类型有3种：eng、user和userdebug。

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