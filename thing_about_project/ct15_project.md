## why -> reason -> result? sure? sure



关于稳定性

#### 1 thread_smv_Receive 有可能修改result发生冲突？

解决：

方法1 ：在main thread 和 thread_smv_Receive thread 中添加状态控制

方法2 ：单一变量在同一个thread中修改 （稳定性会比较好）



#### 2  资源path问题，前期配置的时候没有

解决：fo|相对的路径  -->fail

​	    fo|绝对路径   --->success

log的路径，讯飞也是默认在/sdcard/msc/下面

#### 3 wakeupresource.jet not found?

讯飞的编译出现问题，编译工具用错了

#### 4 read 为0 或者 -1，或者没有voice的情况

这个需要nexel帮忙

#### 5 system() 有时候发送失败

#### 6 call.bnf 相关

只能容纳256个词，语法需要小心

#### 7 语法加载失败？

> 这个是由于残留进程导致的，而残留进程是问题4 导致的。
>

#### 8 误识别问题

#### 9 logout时间太长 

> logout 出现退出时间太长 
>
> --> 退出的时候往msc写是数据
>
> 

#### 10 echovoice 点击退出之后还是会重启，这是为什么？而改动postkey之后，这个问题就解决了。

原因：

方案：结束的时候，如果采用system() 发送keycode，就可以顺利结束。而是用ioctrl的方式则会重新开启新进程，继续任务。非常非常奇怪



#### 11 一次唤醒不成功之后，连续唤醒就会一直不成功。

原因：

解决方案:



#### 12 蓝牙测试，一直无法定频。

问题： 测试的一直提示 内核中CONFIG_BT_HCIUART 在内核中没有开启，会出现 TIOCSETD 设置失败

解决方案：

首先保证wifi是正常的

1、 韩国那边提供一个操作打开蓝牙

>  λ adb shell
> croot@activo:/ # bdt
> set_aid_and_cap : pid 1066, uid 0 gid 0
>
> :::::::::::::::::::::::::::::::::::::::::::::::::::
> :: Bluedroid test app starting
> Loading HAL lib + extensions
> HAL library loaded (Success)
> INIT BT
> HAL REQUEST SUCCESS
>
> 》enable
> ENABLE BT
> HAL REQUEST SUCCESS
> ADAPTER STATE UPDATED : ON
>
>  : unknown command
>
> help lists all available console commands
>
> quit
>
> enable :: enables bluetooth
>
> disable :: disables bluetooth
>
> dut_mode_configure :: DUT mode - 1 to enter,0 to exit
>
> le_test_mode :: LE Test Mode - RxTest - 1 <rx_freq>,
>                                TxTest - 2 <tx_freq> <test_data_len> <payload_pattern>,
>                                End Test - 3 <no_args>
>
> 》dut_mode_configure 1
>
> BT DUT MODE CONFIGURE
> HAL REQUEST SUCCESS