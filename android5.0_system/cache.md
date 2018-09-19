   3.7

writev()这个函数，

Z:\nexell\lollipop-5.1.1_r6-release_3rd\system\core\liblog\logd_write_kern.c

->Z:\nexell\lollipop-5.1.1_r6-release_3rd\system\core\include\log\log.h

->Z:\nexell\lollipop-5.1.1_r6-release_3rd\system\core\include\log\uio.h

->Z:\nexell\lollipop-5.1.1_r6-release_3rd\system\core\liblog\uio.c

```c++

int  writev( int  fd, const struct iovec*  vecs, int  count )
{
    int   total = 0;
    //实际上只写入三次
    for ( ; count > 0; count--, vecs++ ) {
        const char*  buf = vecs->iov_base;
        int          len = vecs->iov_len;
        while (len > 0) {
            int  ret = write( fd, buf, len );
            //write failed
            if (ret < 0) {
                if (total == 0)
                    total = -1;
                goto Exit;
            }
            //finish write
            if (ret == 0)
                goto Exit;

            total += ret;
            buf   += ret;
            len   -= ret;
        }
    }
Exit:    
    return total;
}
```

3.8 ELF文件

参考《elf二进制文件》

3.8.4 与重定位相关的节区信息 ---dynamic段

![1537368885222](E:\mybook\book_principal_work\android5.0_system\ing\%5CUsers%5Celvin%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1537368885222.png)



3.8.5 函数的重定位过程

##### 3.9 Bionic中的Linker模块

ldd 可以区分静态可执行文件和动态可执行文件。

静态可执行文件用在一些比较特殊的场合，例如，系统初始化时，这时整个系统还没准备好，动态连接的程序还无法使用。系统的启动程序Init就是一个静态链接的例子。

3.9.2 可执行文件的初始化。

3.10 调试器--ptrace 和 Hook API

> ptrace函数通常在调试器的调试软件，如gdb中，调试器利用ptrace()函数达到控制目标进程运行的目的。现在国内不少Android安全类软件使用ptrace()函数把自带的动态库"插入"到系统或别的进程中，从而达到监控系统运行的目的。

































