1. make run启动
2. qumu启动远程调试(printf吧，记得emacs-gdb调试)
3. am: 虚拟机，硬件的抽象  klib:静态链接库
	am.h: 提供5组抽象接口
	amdev.h: 声明寄存器等硬件端口
	trm.c:基础图灵机启动，初始化，关机
	mpe.c:实现多线程接口

（**编译项目和qemu的方法:** 项目的makefile指定 NAME，SRCS，INC_PATH，export ARCH    := x86_64-qemu， include $(AM_HOME)/Makefile。然后AM的Makefile逐级include到x86_64-qemu.mk, x86_64.mk, qemu.mk。image作为主的target）


临时:
kernel的mpe_init拉起了smp个线程，参数和入口

elf的_start64会调用_start_c函数，根据is_ap（Application Processors, ap）决定后续执行

#### 问题处理方案
**gdb调试qemu:**
```
Terminal one:
 qemu-system-x86_64 -S -s -serial none -nographic ./build/kernel-x86_64-qemu(镜像文件)
 (qemu)
 
Terminal two:
gdb xxxxx.elf(elf文件)
(gdb) target remote localhost:1234
(gdb) b *0x7c00
```

**smp 不能正确启动多处理器:**
abstract-machine 和 xv6 都通过读 MP tables 来得到 CPU 核心数的值，这个值对应 QEMU 中的 sockets 数量。如果发现 AbstractMachine 只找到了一个核，在 scripts/platform/qemu.mk 里面的 QEMU_FLAGS 里把 -smp 设置成 -smp “cores=1.sockets=$(smp)” , smp设置为多个即可。
