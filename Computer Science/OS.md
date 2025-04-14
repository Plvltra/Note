https://jyywiki.cn/OS/2024/
#### 并发
- 操作系统的应用程序是状态机, 运算是状态机状态转移(除了syscall)
- 中断: 打断某个进程/线程状态机的执行流(比如可以是输入某个按钮)。程序可以通过设置寄存器flag关闭中断响应(这种属于特权指令，只有OS有这种权限)
- x86内存模型: 不同cpu先write到cache, 再store到memory, read一定能获取到store memory的值
- Peterson算法: 假设了load和store操作的原子性，同时只能用于两个程序并发的同步
- volatile: 告诉编译期取消优化，例如对某个变量for循环加十，如果优化等级过高或对寄存器中的值+10导致多线程改变该值结果错误。volatile会强制去内存中读取值增加

#### 第6-7章: 并发：互斥
###### 1. Peterson算法与atomic
Peterson算法: 只假设load和store实现互斥 缺点: 理解太复杂
atomic指令: 汇编指令前加lock或者Test-And-Set (TAS), Compare-And-Swap (CAS), COMPXCHG (Compare-And-Exchange)等指令实现原子操作

###### 2. 操作系统内核中用的自旋锁实现
实现并发自旋锁的主要原因: 操作系统存在中断，一旦中断处理程序也要用spin lock住的资源就会死锁
目标: spin lock前per cpu的保存中断状态，关闭中断，自旋，spin unlock后恢复中断状态

###### 3. LinuxRCU避免锁技巧
Linux内核无锁操作技巧: Read-Copy-Update, 假设写远比读少，拷贝修改数据结构后，切换指针回去

###### 4. Mutex
自旋锁的缺点: 
1. 其他程序在空转等待占用cpu，本来可以处理其他任务
2. 应用程序不能关中断，导致持有自旋锁被切走需要锁换上来，100%浪费

解决方式: 通过syscall让操作系统调度 syscall(SYSCALL_lock) syscall(SYSCALL_unlock)	
Futex: 先自旋，自旋获取不成再系统调用

#### 第8章: 调试技巧
###### 1. 调试理论
编码缺陷:fault -> 状态机状态异常: error  -> 软件系统不满足需求: failure
调试理论: 给定一个failure, 可以二分定位到第一个error状态, 此时的代码就是fault

###### 2. 不同工具检查状态
ssh -v 检查日志
gcc -v 打印过程
(提供打印中间状态，即-v选项是linux程序的一项原则)
make -nB查看完整build历史
perf 采样状态机
strace追踪系统调用过程

###### 3. 调试理论推论
需求 → 设计 → 代码 → Fault → Error → Failure

1. **写好代码**: 不要在写代码的时候忘记需求和设计(保证代码能映射需求)
	- 不言自明 (Self-explanatory)
	    - 能通过字面知道需求 (流程)
	- 不言自证 (Self-evident)
	    - 能通过字面确认代码和需求一致

2. **做好测试**：未测代码永远是错的(保证fault都能变成error)
	- Small Scope Hypothesis
1. **多写断言**：把代码中的 “隐藏性质” 写出来(保证error和failure距离近)
	- Error 暴露的越晚，调试越困难
	- 追溯导致 assert failure 的变量值 (slice) 通常可以快速定位到 bug

#### 第9-10章: 并发：同步
1. **同步**：只有互斥不够，需要能够给控制并发，使得 “两个或两个以上随时间变化的量在变化过程中保持一定的相对关系”
2. 生产者-消费者问题与**条件变量**
	- 条件变量万能模版
	```
	mutex_lock(&mutex);
	while (!COND) {
	  wait(&cv, &mutex); // 此处wait会先释放锁以便让其他线程进入临界区修改共享数据
						 // 然后进入休眠等待被唤醒，wait的后面有执行加锁(访问临界区)
						 // 唤醒后自旋判断是否COND同步条件是否满足
	}
	assert(cond);
	
	...
	
	mutex_unlock(&mutex);
	```
	- 唤醒条件变量
	```
	cond_signal(&cv);
	cond_broadcast(&cv);
	```
3. 同步机制的应用(通用并行)
	- 生产者消费者的普适性: 任何计算任务可以拆解成有向无环图，生产者向任务队列添加任务，消费者处理任务。
	- 为每个节点设置一个条件变量
		- - (u,v)∈E 表示 v 要用到前 u 的值
		- v 能执行的同步条件：u→v 都已完成
		- u 完成后，signal 每个 u→v

4. **信号量**
	- 想法起源: 创建一个锁时，立即 “获得” 它。release线程释放它，acquire线程获取它，实现同步
	- 实现计算图
		- 为每一条边 e=(u,v) 分配一个互斥锁 
		- 初始时，全部处于锁定状态
		- 对于一个节点，它需要获得所有入边的锁才能继续
		    - 可以直接计算的节点立即开始计算
		- 计算完成后，释放所有出边对应的锁
	- mutex 是信号量 n=1 的特殊情况, P尝试获取1单位，V释放1单位
	- 两种典型的应用
		- 实现一次临时的 happens-before
		- 管理计数型资源 (停车场里的车不能超过 n 个等)
5. 信号量、条件变量实现同步对比
	- 条件变量
		- **万能**：适用于任何同步条件
		- 不太好用：代码总感觉不太干净
	- 信号量
		- 在符合条件下干净、优雅(**信号量不总是 “优雅”**)
	- 条件变量能轻易实现信号量，信号量确没法实现条件变量, 具有**不对称性**
	- 例如哲学家就餐问题: 使用信号量实现很容易出现拿起一半筷子死锁，但是使用条件变量就比较简单(因为只有满足条件才会同时拿起筷子)

#### 第11章: 真实世界的并发编程
1. 并发编程框架实例
	 - 解决回调地狱: Promise描述动态计算图(Web2.0 javascript的异步)
	```javascript
		// then表示回调
		useEffect(() => { 
			fetch(`/api/localhost/?action=demo&path=${path}`).
				then(response => response.json()).
				then(fetchedData => setData(fetchedData)) 
		}, []);
		
		// Promise.all表示回调结束后继续回调
		Promise.all([ 
			fetch(...).then(...), 
			fetch(...).then(...), 
			fetch(...).then(...), 
		]).then( 
			// succeeded 
		).catch( 
			// error handling (catches exceptions in the fetch) 
		)
	```
	- 高性能科学计算的并发: OpenMP
2. 数据中心中的并发编程
	1. Go和Goroutine
		- 协程的问题: sleep (I/O) 时，所有协程都 “卡住了
		- **Goroutine实现**: 概念上是线程，实现是线程和协程的混合体
			- 每个 CPU 上有一个 Go Worker，运行协程
			- 协程执行 blocking API (sleep, read)
			    - 偷偷调用 non-blocking 的版本
			    - 成功 → 立即继续执行
			    - 失败 → 立即 yield 到另一个需要 CPU 的 goroutine(等指定时间后默认sleep完成)
	1. Go语言中的同步与通信
		- 共享内存 = 万恶之源
		- 管道通信:  Channels in Go

3. 人工智能时代的并发编程
	- **SIMD 指令** (Single Instruction, Multiple Data)
		- Intel 处理器的 SIMD 经历了 MMX指令集 (MM 寄存器)、SSE指令集 (扩展为 XMM 寄存器)、AVX指令集 (扩展为 YMM 寄存器)、AVX512指令集 (扩展为 ZMM 寄存器) 的发展
	- **SIMT 模型** (Single Instruction, Multiple Threads)
		- 没有 per-thread的program counter，而是一个线程束共享一个 program counter
		- 一个 program counter，控制 32 个执行流同时执行
		- 执行流有独立的寄存器, xy,zx,yz 三个寄存器用于标记 “线程号”, 然后堆海量的线程

#### 第12章: 并发 Bugs
1. 死锁(**有通解**)
	给所有锁编号，严格按照从小到大的顺序获得锁(任意时刻，总有一个线程获得 **编号最大的锁**, 这个线程总是可以运行完)
2. 数据竞争(难解决)
	- 可能是Atomicity violation，本应原子完成不被打断的代码被打断
		- “ABA”: 代码被别人 “强势插入”
		- 共享状态中途被人更改
	- 可能是Order violation，本应按某个顺序完成的未能被正确同步
		-  “BA”: 事件未按预定的顺序发生
			- concurrent use-after-free(本应使用完同步其他线程free, 同步未达成)

#### 第13章: 应对 (并发) Bugs
1. 通用调试工具
	- Linux Kernel Lockdep (运行时建立锁的依赖关系来检查死锁, 锁是根据某行来设置的)
	- ThreadSanitizer(不同线程unlockA happens-before lockA提供了保证, 查未得到保证的同一块内存data race) 
	- AddressSanitizer, UndefinedBehaviorSanitizer....(思想都是实现了对某一类specification的检查)
2. 适用于Lab1的简化specification检查
	- Stack Guard(栈顶栈底加一段canary内存)
	- 低配Lockdep(对于简单的代码自旋超过一微秒, 即可认为死锁)
	- 低配AddrSanitizer(对分配和释放内存涂色，check涂色状况)
	- 低配ThreadSanitizer(在一段锁中观察一次临界内存等一小段时间再观察临界内存，看看是否一致，否则有其他地方竞争写了)
	- 低配SemanticSanitizer(写一些指针位置assert宏, 防止指针的值很奇怪)

**真正的编程: 把对程序的预期Specification表达在程序中**

#### 第14章: 操作系统上的进程
1. fork会将"当前状态机的全部状态"复制一份，除了rax返回值不一样
2. execve 将当前进程重置成一个可执行文件描述状态机的初始状态, 允许对新状态机设置 argv 和环境变量 envp
3. _exit立即摧毁状态机，允许有一个返回值
	- glibc提供exit(0),会调用atexit。
	- _exit(0)执行exit_group终止进程和所有线程 (linux提供, 不会调用glibc atexit)
	- syscall(SYS_exit, 0) (linux提供, 不会调用glibc atexit)

#### 第15章: 进程的地址空间
1. mmap/munmap可以在进程状态机上增删一段可访问内存, mprotect可以修改映射权限
    - (mmap可以瞬间实现大量内存分配，还可以讲文件映射到内存)
    - vdso, vvar等特殊段: 操作系统共享内存页给进程，避免进程获取时间等简单信息还要syscall的开销(例如gettimeofday()的实现)
2. 修改进程内存: /proc/[pid]/mem文件是进程运行时内存，修改内容
    xdotool/ydotool向进程发送GUI事件，evdev按键显示脚本
    修改进程对时间的感知: 修改进程感知时间变化的函数所在的内存，换成自己的
    live patching: 同上，运行时修改函数所在的内存

#### 第16章: 系统调用和 UNIX Shell
1. 操作系统对象
	- **操作系统 = 操作系统对象 + API    操作系统对象 = 进程间共享空间**
	- 进程通过syscall将操作系统对象纳入本进程内存空间，以实现进程间通信
	- 操作对象实例： 文件描述符(问文件或其他输入/输出资源的"指针")，UNIX 管道(一端进一端出，有阻塞同步效果。管道常见用途: 创建管道后fork, 子进程拥有了读写端口描述符。可以关掉读或写, 实现IPC)
2. 操作系统Shell
	- 基于文本替换的快速工作流搭建
		- 重定向: cmd > file < file 2> /dev/null
		- 顺序结构: cmd1; cmd2, cmd1 && cmd2, cmd1 || cmd2
		- 管道: cmd1 | cmd2
		- 预处理: $(), <()
		- 变量/环境变量、控制流……
	- Job control
		- jobs, fg, bg, wait
	- 建议man sh用半小时学习shell使用指南
3. UNIX世界全部机制
	进程管理(fork, execve, exit),
	内存管理(mmap, munmap, mprotect)
	文件管理(open, close, read, write, lseek, dup) 
	进程间通信(pipe, wait)

#### 第17章: 系统调用和 UNIX Shell
1. libc 简介
	- 系统调用是地基，C语言是框架
	- C 语言——世界上 “最通用” 的高级语言，是系统调用的一层 “浅封装” ，比C++具有更强的移植性
	- [musl](https://musl.libc.org/) 没有glibc的优化，更适合学习用

#### 第18章. Linux 操作系统
1. 最小 Linux
	- init进程(1号进程)执行initramfs(提供的init脚本)。先执行了挂载busybox中的所有命令，之后初始化文件系统并mount、exec swithc_root /newroot /init(swithc_root内部调用pivot_root)切换文件系统，使得df /能查出文件系统
	- linux上 /sbin/init -> /lib/systemd/systemd, 上述exec swithc_root /newroot /init也就是将init进程切换执行/fsroot/init, 用自定义的脚本替代系统systemd逻辑
	- /etc/fstab是用于供systemb修改文件系统的固定配置文件
	- /fsroot/init: 替代systemd的样例，展示在我们自定义的systemd中可以配置网卡，启动HTTP服务的潜能

2. 构建应用程序的世界 
	- wsl: 所有程序运行在 os+api上，那么只要将linux程序的syscall api翻译成windows系统的同义行为即能实现
	- 还有wine(linux subsystem for windows)
	- 甚至还有苹果rosetta这种翻译不同指令集软件的工具

#### 第19章. 可执行文件和加载
1. 可执行文件: 一种程序初始内存状态的描述数据结构
2. **静态链接**做的事:
	1. 第一个pass: 分别合并.text, .data, .bss中的代码, 把 sections “平铺” 成字节序列。
	2. 第二个pass: 确定所有符号的位置解析, 将所有重定位符号赋予相对于当前位置的偏移
3. shebang与loader
	- shebang: #!在文本第一行可以作为文件解释器，例如#!./exec会调用我们的exec程序加载FLE
	- loader: 课程实现的execve加载elf程序(原理: readelf和objdump解析elf文件段，将对应的段mmap到内存空间, 根据SystemV ABI约定创建初始栈和寄存器布局)

#### 第20章. 动态链接和加载
1. 动态链接
	1. 动态链接有点:  大项目可拆分, 更新方便不用重编, 减少运行时内存(所有程序公用一个libc.so)
	2. 面临的问题: 编译时不知道一个编译单元未定义符号运行时地址
		- 方案一: 在加载时完成重定位(不能延迟绑定)
			-  加载 = 静态链接
			-  省了磁盘空间，但没省内存
		- 方案二: 编译器生成**位置无关代码**
			-  加载 = mmap(多个进程映射同一个libc.so，内存中只需要一个副本)
			-  但函数调用时需要额外一次查表
2. 加载100次同一个1GB的静态链接ELF内存变化: free内存减少，buff/cache增长1GB(由于Copy-on-Write对代码段的复用)
3. 动态链接的实际做的事(以调用putchar)
	- 静态链接: 直接寻址。编译时生成一个占位符，链接阶段填入putchar的实际地址
	 - 动态链接: 间接寻址。编译时编译单元生成一个putchar符号对应8个字节, 生成汇编
		call \*putchar(%rip)，call \*putchar(%rip)相当于调用putchar符号相对于当前rip偏移量所在地址存放的值(也就是putchar符号对应地址的值)。链接时向putchar符号填值。我们发明了GOT
1. 动态链接实现的额外细节
	- 对于动态链接库的函数, 在编译时的32位rip相对偏移可能不够(本文件被加载到0x0000555555554000, 动态链接库被加载到0x00007fffffff6000差距超出32位)。如果是静态链接的函数，则call直接跳转。如果链接时发现动态符号，创建一个局部plt函数(32位以内)，用于二次跳转到GOT的符号所在地址
	- 对于动态链接的数据, 也存在编译时留下的32位跳转空跳不过去的问题。解决方案: extern int a 被编译器保守地视为可能的动态链接，在GOT会有一个距离当前rip 32位以内的地址存放目标变量的值，用于间接跳转。extern int __attribute__((visibility("hidden"))) a 被视为本库的内容，直接32位内跳转

#### 第21章. 系统调用、中断和上下文切换
1. 动态链接续
	- 动态链接数据访问优化: -fPIE编译生成的会将动态链接数据搬入可执行文件(重定向表对应类型R_X86_64_COPY)，不用二次跳转。-fPIC编译生成的动态链接数据不会使用这种优化(重定向表对应类型R_X86_64_GLOB_DAT)。
	- 阅读ld.so manuel, 了解其做的事情
		- LD_PRELOAD:  ld.so提供的机制，通常用来library interpositioning(库打桩), 替换官方库的视线改为自己的实现
		``` c
		// LD_PRELOAD hook malloc example
		void* malloc(size_t size) {
		    write(STDOUT_FILENO, "hello\n", 6);
		    void* (*original_malloc)(size_t) = NULL;
		    original_malloc = dlsym(RTLD_NEXT, "malloc");
		    return original_malloc(size);
		}

		// compile
		gcc -shared -fPIC -o libmalloc.so malloc.c -ldl
		gcc -fPIC -c main.c -o main.o
		gcc -o main main.o -L. -lmalloc
		// execute
		LD_LIBRARY_PATH=.:$$LD_LIBRARY_PATH LD_PRELOAD=./libmalloc.so ./main
		```
2. syscall 指令
	- syscall指令的内容: 保存rip和rflags(好像是当前中断状态)，rip设置为内核中的系统调用入口地址。 设置ss,cs为内核态权限，rsp切换到内核栈(IA23_LSTAR)。执行完syscall后再执行sysret逆操作
	- 虚拟地址介绍Complete virtual memory map: https://www.kernel.org/doc/html/v6.3/x86/x86_64/mm.html
3. 中断机制和上下文切换
	- 中断本质: 保存被中断状态机的寄存器到内存，中断切换调度, 加载新状态机恢复寄存器。
	- **thread-os.c实现中断调度解读**
		- 用户线程通过kcontext获得初始寄存器状态
		- 不同状态机(操作系统/示例线程)主动或被动interrupt(int指令)-> 通过IRQS(IDT_ENTRY)注册的入口进入IRQS(IRQ_DEF)对应的中断号处理逻辑 -> trap按顺序保存寄存器 ->\_\_am_irq_handle(on_interrupt被调用时保存当前状态机的寄存器)->\_\_am_iret(恢复新切换状态机的寄存器)
			- \_\_am_irq_handle: 将栈顶解读为trap_frame类型用于保存旧状态机寄存器。 硬件(int指令)保存 uint64_t rip, cs, rflags, rsp, ss;到栈上, abstrace machine代码(IRQS(IRQ_DEF)展开的push)保存uint64_t irq, errcode
			- \_\_am_iret: 恢复寄存器，其中rip，rsp需要同时恢复，用一条iretq实现
	- 中断可以理解为强制 “插入” 的 syscall。前者进程被动响应信号handler，保存寄存器状态，被操作系统调度。后者进程主动发生一系列状态保存，切换等

#### 第22章. 进程的实现
1. 进程与虚拟地址空间
	- 虚拟内存与分页机制:
		- x86的普通实现
			- 要解决的问题: 将虚拟地址空间映射到物理页上
			- 以32位为例:
				- 需求:硬件好实现，快速寻址分成，将前20位的页地址映射到物理内存页
				- 做法:使用2级1024叉树，前20位分成10位 + 10位，用一页(4KB = 2^10个指针)存一级索引，每个一级索引指向一页的二级索引(2^10个指针, 指向物理地址)
			- 64位: 只有两级页表不够用，所以有PML4/PML5等多级页表
			- CR3寄存器: 指向一级页表, 每个进程有自己的CR3(每个进程都有自己的页表结构)
			- TLB缓存: 不用每次查虚拟地址都必须跳转，先查TLB缓存
		- MIPS的实现: TLB cache miss跳转到操作系统代码，由代码软实现决定vpn->ppn的映射	
		- Inverted Page Table的实现: 存一个全局的Hash Table, 映射(VPN, pid)->PPN
			- 缺点: TLB miss找Hash Table可能被恶意程序冲撞的厉害
	- Demand Paging机制:
		1. 用户进程只是load/store一个地址
		2. 操作系统维护f数据结构映射并且知道对应的页是否分配
		3. 如果发生Page Fault: 修改虚拟页到物理页的映射数据结构，加载页
		好处: 只用加载局部物理页
	- Copy-on-write机制:
		1. fork() 后直接把父子进程地址空间标记成只读, 虚拟页映射向共同物理页
		2. 当进程发生写操作复制一份页并修改映射，当对页面的引用计数为0后，最后一份只读副本也变成可写

2. **UNIX 和 xv6(重要)**
	**本课程前20节是应用视角的操作系统(如何使用)，后面是操作系统机制的实现**
	1. 学习操作系统机制并自己实现 xv6: a simple, Unix-like teaching operating system: https://pdos.csail.mit.edu/6.828/2024/xv6.html
	2. unix介绍 A commentary on the sixth edition unix operating system:  http://www.lemis.com/grog/Documentation/Lions/

#### 23. 处理器调度
计算机系统设计中的一个重要主题就是**机制和策略的分离**
1. 上下文切换机制回顾：
	- xv6: 进程和操作系统共享两个页面(trampoline和trapframe)
		- trampoline: 一段简短的，用于状态准备跳转的代码(0x3ffffff000)
		- trapframe: 寄存器的快照保存
	- xv6的上下文切换方式(以xv6切换进入第一个init进程为例):
		- make qemu-gdb启动qemu等待连接, gdb-multiarch -x init.py连接启动调试 
		- xv6启动流程: 一小段boot代码 -> entry.S -> start.c 
		- 下面是开机到进入init.c进程发生的事:
			```
			一段不知道哪里的0x0开始的代码(直接走物理内存, 没有页表), 猜测是proc.c的initcode?
			->entry.S
			->start.c(配置好用户进程的页表, mret)
			->这时候的0x0已经装上了inticode.S, 从0x0开始执行，跑inticode.S start段执行exec init
			->ecall开始执行trampoline.S
			->trampoline.S执行uservec的csrw satp, t1切换至内核页表
			->trampoline.S然后不返回的jump到trap.c的usertrap()(usertrap好像保存当前寄存器到trampoline前一帧?)
			->trap.c->syscall.c->sysfile.c(sys_exec: 按照图示配置出进程初始地址空间)->exec.c(exec加载init进程, 做的和我们的elf加载大体一致)
			->之后进入trap.c usertrapret(), 最后一行执行到trampoline.S的userret, 设置用户进程的页表和恢复其他寄存器
			->trampoline.S最后的sret将pc置为0, 从0x0开始执行init.asm(此时init进程text,data已经在用户进程页表映射的页下就绪)
			```
		- (qemu) info mem查看qemu内存分布, info registers查看寄存器
		- 我们构造了init进程的内存布局，只需要切换vr眼镜切换计算机视角到init内存分布。这个例子证明了**操作系统在启动完第一个进程后，就变成了一个中断处理程序**
2. 处理器调度策略:
	- MLFQ:
	- Complete Fair Scheduling:
		- sched_latency(用户定义的调度器目标延迟,即每个任务在此时间范围内至少能够获得一次CPU执行机会), n(n为当前的任务数), 时间片time_slice(sched_latency / n)
		- 每个任务有自己的权重, 实际分配的时间片是加权划分的(例如sched_latency=48ms, A权重1024, B权重3072, 则A的时间片是12ms，B的时间片是36ms)
		- 任务累加的virtual runtime时还是不带权重，选取消耗vruntime最小的任务优先调度
		- 应对I/O与休眠: 为防止任务休眠一段时间被唤醒后vruntime太小饿死其他任务, 任务从休眠或I/O中返回时，vruntime会被设置为当前红黑树中的最小vruntime值
		- 使用红黑树提升vruntime查找效率	
3. 处理器调度实践:
	- 单处理器调度:
		- 低优先级任务持有高优先级任务锁的情况(优先级反转)
		- 解决方案: 优先级继承, 持有mutex的任务会继承优先级最高的等待mutex任务(不总是有效)
	- 多处理器调度:
		- 不能简单地分配线程到处理器(容易空等), 也不能简单地谁空丢给谁(cache/TLB miss)
		- NUMA, 异构处理器，多用户，分布式系统，异构计算... 每种都很难调度


#### 24. 操作系统世界小结
1. 故事
	- 一切皆为状态机，syscall是从外部改变了状态机状态
	- 脑洞大开: 元胞自动机也是一种状态机，我们可以通过设置一个“特殊格子”来作为syscall的交互
2. 推论
	- 方法论![[tty-session.webp]]
		- ”最小” 线
			- C 程序 (hanoi.c)
			- 汇编代码 (minimal.S)
			- 操作系统内核 (thread-os)
		- “实际” 线
		    - 最小 Linux (initramfs)
		    - musl libc; ELF 文件
		    - 金山游侠、按键精灵、变速齿轮
		        - 在与实际程序交互的过程中理解操作系统
	- 状态机视角推论
		- 我们能够观察状态机执行
		- 甚至能记录/改变状态机执行
			- Record-replay
			- 隔一段时间中断程序，记录指令和状态迁移，得到性能摘要
			- Mdele Checker和Verifier(可以做状态合并减少状态)
3. 应用
	- 例子: 以Ctrl+C可以中断进程为例
		- tmux中ctrl+c是否会杀死全部进程?(机制1: 终端和Shell, vscode, tmux, ttyd是terminal(终端设备)，连接到interactive shell)
		- ctrl+c发出信号(机制2: 信号通常发给进程组setpgid, getpgid)
			![[Pasted image 20250114180812.png]]
		- 进程自身注册信号处理逻辑
			- kill本意是发送信号, kill -9发送`SIGKILL`信号

#### 25. 1-Bit 的存储
1. 磁存储
	- 磁带: 一维纸带上沾满磁性颗粒, 根据颗粒的上下表示01，读写头经过读取内容
		- 可以缠成卷变成2D存储。成本低, 高信息密度, 存在机械部件可能丢失。无法随机读写
	- 磁盘:
		![[Disk.png]]
		- 由磁卷成盘，多个磁盘用轴串联成2.5D。需要随机读写时，靠磁盘转到特定位置由读写头
		- 成本低, 高信息密度, 随机读取受限于机械转速(转到读写头才能读取)
	- 软盘
2. 坑存储(光)
	- 反射平面挖上粗糙的坑过，激光扫过就可以读取
		- 容量大，拷贝复刻成本低，但是只读(激光扫过填坑困难)
3. 电存储
	- Flash Memory(闪存)
		- 批量生产低价，高容量，随机读取告诉(电速)
		- 致命缺点: 放电数千/数万次以后,就永远是充电状态了
	- U盘, SD 卡, SSD 都是 NAND Flash
		-  SSD 里藏了一个完整的计算机系统(FTL: Flash Translation Layer, 用软件使写入变得 “均匀”)
			- SSD 的 Page/Block 两层结构
			    - Page (读取的最小单位, e.g., 4KB)
			    - Block (写入的最小单位, e.g., 4MB)
			    - Read/write amplification (读/写不必要多的内容)
			- Copy on write

#### 26-输入输出设备原理
1. I/O设备原理(I/O 设备 = 一个能与 CPU 交换数据的接口/控制器)
	- Interface=一组寄存器(保存Status, Command, Data)
	- 给设备寄存器赋予一个内存地址, CPU 可以直接使用指令 (in/out/MMIO) 和设备交换数据

2. 串口、键盘、磁盘和打印机
	- 串口: 代表COM1, CPU向特定寄存器偏移outb或者inb可实现写入读入

3. 3个特殊的I/O设备(总线、中断控制器、DMA)
	- 总线: 一个特殊的I/O设备
		- CPU连接总线，总线寻址IO设备寄存器, 实现CPU与IO寄存器的数据传输
		- 总线对CPU提供了设备的虚拟化
		- PCI总线桥接USB总线, USB也是树状结构,对PCI提供虚拟化(测试lsusb,lspci )
	- 中断控制器:
		- CPU只有一根中断线
		- 中断控制器收集各个设备，按优先级发送给CPU，执行对应中断号的处理
			APIC (Advanced Programmable Interrupt Controller)
			local APIC: 中断向量表, IPI, 时钟, ……
			I/O APIC: 其他 I/O 设备
	- DMA:
		- 解放cpu算力，专门执行memcpy的cpu
	- 只要满足了和cpu的通信协议, 可以扩展出任何的I/O设备
4. GPU和加速器

#### 27-文件和设备驱动
1. 文件和文件描述符
	-  指向操作系统对象的"指针"
	- 用户态进程无法直接访问操作系统对象，通过syscall(open/close, read/write) + 向操作系统提供文件描述符实现
	- 操作系统内部维护(进程id, 文件描述符)到操作系统对象的映射表, 用户进程发起syscall时查表执行操作	
	- 更多细节
		- 文件访问时的offset管理
			- read/write 操作系统自动维护offset(不暴露给用户), lseek系统调用可以修改offset(减少向用户暴露的细节，简单易用)
		- 文件描述符在 fork 时会被子进程继承
			- Linux系统认为父子进程向一个文件写log是常见的场景， 选择共享offset(不共享会导致一个被另一个覆盖)
			- 操作系统的每一个 API 都可能和其他 API 有交互, 导致了设计的复杂性
				- open 时，获得一个独立的 offset
				- dup 时，两个文件描述符共享 offset
				- execve 时文件描述符不变
2. 设备驱动程序
	- 设备驱动程序就是一个struct file_operations的实现，把系统调用"翻译"成设备能听懂的数据
	- everything is a file有时候不是这样, 设备不仅仅是数据，还有复杂的操作
		- 实现方法1: 控制作为数据流的一部分(ANSI Escape Code)
		- 实现方法2: 提供一个新的接口 (request-response) √
			- **ioctl系统调用**(例如: KVM Device 实现虚拟化可以靠 ioctl(kvm, KVM_CREATE_VM, 0)创建VM, ioctl(kvm, KVM_CREATE_VCPU, 0)创建vCPU等实现)

#### 28-文件系统
1. 文件系统
	- Unix一切都在/中
		- 初始时只有 /dev/console 和几个文件。/proc, /sys, /tmp都是通过mount挂载创建出来的
		- 如何挂载一个filesystem.img?
			- 操作系统会给镜像创建一个 loopback (回环) 设备, mount loopback设备
	- 链接方式
		- 硬链接: 允许一个文件被多个目录引用, 不同目录链接同一个inode, 修改对所有生效。 引用为0时清除本体。不能硬链接目录(会递归)和跨文件系统
		- 软链接: 相当于快捷方式，没有任何限制, 操作系统访问软链跳转到文件本体。(即使没有本体也可以先创建软链接)
2. FAT 和 UNIX 文件系统
	- 文件系统: 一个**硬盘上的"数据结构"**。之所以要有文件系统而不用数据结构, 主要是数据结构存储在Random Access Memory(缓存等硬件构造了内存可以随机访问这一假象)。而文件系统存储在block device一次读取只能用bread/bwrite读取一块。**用 bread/bwrite 模拟 RAM → 严重的读/写放大**
	- FAT: 将数据块指针链表集中存放在文件系统的某个区域(针对当时的workload够用了)
		- 以普通文件的方式存储 “目录” 这个特殊数据结构的内容
		- 优点：局部性好, 访问一个数据块加载所有指针链表
		- 缺点：集中存放的数据损坏将导致数据丢失(可以通过备份n份解决)
		- 性能: 小文件合适, 4 GB 的文件跳到末尾 (4 KB cluster) 有 2^20次 next 操作
	- ext2
		- 集中存储文件/目录元数据
		- 为大小文件区分 fast/slow path
			- 对于文件前11个块的访问: inode Direct Blocks直接存储数据块id
			- 对于文件靠后的数据访问: 通过三级页表跳转
				- 例如访问(12, 256+12)的数据块先访问Indirect blocks拿到一级页表数据块, 再到以及页表对应条目拿到数据块位置, (256+12, 256^2+256+12)走二级页表
				- 对于大文件随机访问也最多5次bread
		- 目录文件与 FAT 本质相同：在文件上建立目录的数据结构。存储inode到文件名的映射
		- 参考:https://www.jianshu.com/p/3355a35e7e0a https://www.cnblogs.com/alantu2018/p/8459284.html

#### 29-持久数据的可靠性
1.  RAID(一种虚拟化: 把多个不可靠磁盘虚拟化成一个可靠的磁盘)
	 ![[RAID.png]] 
	 虚拟块号到 (磁盘, 块号) 的 “映射”。类似于虚拟内存的思想，提供一种映射，把virtual block映射到physical block
	 - 不同物理磁盘读写可以高速**并行**；存储 >1 份即可实现数据**容错**
	- raid0: 虚拟块一分为2, 交替放在两个物理盘上, 没有数据备份。 读/写速度 x 2
	- raid1: 虚拟块在两个硬盘各保持一份。读速度 x 2, 写速度不变
	- raid5:
		- 假设同一时刻只会有一个硬盘出问题
		- 只用 1-bit 的冗余，恢复出一个丢失的bit(假设有5块盘, 其他4块存储数据，最后一块存储4块盘的校验位(其他几块的异或结果)。任意一个盘bit损坏可以通过其他几块异或计算恢复)
		- 校验块交替存在各个盘(负载均衡, 防止存在一个盘, write过于频繁)
2. 分布式
	- GFS分布式文件系统(文件系统就相当于一个磁盘)
	- 大致是将整个互联网上的计算机通过raid当成一个大虚拟文件系统? 结合Map+reduce实现高度并行的数据处理? 
	- 云盘(数据中心存储): 一个分布式的RAID
3. 崩溃一致性和redo日志
	- 起因: 磁盘的bwrite, bread, bflush并不提供多块读写的"all or nothing"支持
		- 崩溃时对文件写入(如文件amend涉及到修改inode块, 修改bitmap块, data块修改)可能只发生了部分, 导致文件系统状态不一致(如bitmap被修改, inode还没有记录导致bitmap的块永远无法被利用)
		- File System Checking(FSCK)可以开机时遍历文件, 可以部分恢复文件系统一致性
	- 实现原子的文件系统修改
		- 先用append-only记录历史操作Journaling(相当于redo log), 等待落盘, 再更新数据结构。如果数据结构更新不一致，继续做未完成的, 或清理已完成的
		- Journaling (TX: transaction)
			1. 定位到 journal 的末尾 (bread)
			2. bwrite TXBegin 和所有数据结构操作
			3. bflush 等待数据落盘
			4. bwrite TXEnd
			5. bflush 等待数据落盘
			6. 将数据结构操作写入实际数据结构区域
			7. 等待数据落盘后，删除 (标记) 日志
		- Journaling性能优化
			- 批处理: 多个journal transaction合批, 减少log大小, 定期 write back journal
			- Checksum (ext4): 不再标记 Tx, 直接标记 Tx 的长度和 checksum
			- Metadata journaling (ext4 default) : 只对 inode 和 bitmap 做 journaling, 数据段丢了就丢了
				- 导致应用程序丢数据测试: https://zhuanlan.zhihu.com/p/25188921

#### 30-课程总结
1. 操作系统提供了程序执行新的轮廓
	- 打印 Hello World
		- 应用程序 (app) → 库函数 → 系统调用 → 操作系统中的对象 → 操作系统实现 (C 程序) → 设备驱动程序 → 硬件抽象层 → 指令集 → CPU, RAM, I/O设备 → 门电路
	- 编译优化的边界: syscall(凡是参与syscall的数据, 可能设计当前状态机外部的, 不能参与优化)
2. 操作系统主题扩展
	- 并发: 走向分布式系统级别的并发
	- 虚拟化: 重新理解操作系统设计, see also: Monolithic Kernel, Microkernel, Exokernel, Unikernel(没有人规定操作系统里一定要有 “文件”、“进程” 这些对象)
	- 持久化: 文件系统没能解决的需求(大量订单、用户、网络 + 非目录遍历性质的查询)
3. 关联学科
	- Computer Architecture: 计算机硬件的设计、实现与评估
	- Computer Systems: 系统软件 (软件) 的设计、实现与评估
	- Network Systems: 网络与分布式系统的设计、实现与评估
	- Programming Languages: 状态机 (计算过程) 的描述方法、分析和运行时支持
	- Software Engineering: 程序/系统的构造、理解和经验
	- System/Software Security: 系统软件的安 (safety) 全 (integrity)

- 课程升华: **Remember brick walls let us show our dedication. They are there to separate us from the people who don't really want to achieve their childhood dreams.** [Randy Pausch Last Lecture](https://www.youtube.com/watch?v=ji5_MqicxSo)