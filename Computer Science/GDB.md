#### 技巧
- 终极gdb: http://www.skywind.me/blog/archives/2036
- gdb cheat sheet: 一页纸包含gdb常用命令 https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf
- gdb tui模式: 文档介绍https://sourceware.org/gdb/current/onlinedocs/gdb.html/TUI.html
	(处理wsl tui无法启动 https://github.com/microsoft/WSL/issues/8516)
- 使用脚本调试: gdb -x XXX.py, 以gdb调试qemu为例
``` python
import os
import gdb

def on_quit():
    gdb.execute('kill')

gdb.events.exited.connect(on_quit)

gdb.execute('target remote localhost:1234')

am_home = os.environ['AM_HOME']
path = f'{am_home}/am/src/x86/qemu/boot/boot.o'
gdb.execute(f'file {path}')

gdb.Breakpoint('_start')

gdb.Breakpoint('start32')

gdb.execute('continue')
```
- gdb多线程调试: https://zhuanlan.zhihu.com/p/673125330
- gdb调试复杂数据结构: https://zhuanlan.zhihu.com/p/673521127
- gdb反向调试:  https://zhuanlan.zhihu.com/p/673279895 ([record replay](http://link.zhihu.com/?target=https%3A//github.com/rr-debugger/rr), [record, replay](https://zhuanlan.zhihu.com/p/364075104 ) rr工具更加好用)
- 使用回车重复上一条指令
- 查看进程的内存地址分布: gdb starti 然后info inferiors查看进程的pid, 然后可以用pmap
- gdb with python scripting: https://zhuanlan.zhihu.com/p/152274203 ,(需要安装时使用--with-python配置)
``` python
### 自定义类pretty printer打印方式
# typedef struct task {
#     union {
#         struct {
#             const char  *name;
#             void        (*entry)(void *);
#             void        *arg;
#             Context     context;
#             task_t      *next;
#             char        end[0];
#         };
#         uint8_t stack[8192];
#     } data;
# } task_t;
class TaskPrinter:
    def __init__(self, val):
        self.val = val

    def to_string(self):
        return 'task_t'

    def children(self):
        # 访问并生成所有成员变量
        yield 'name', self.val['data']['name'].string()
        yield 'entry', self.val['data']['entry']
        yield 'arg', self.val['data']['arg']
        yield 'context', self.val['data']['context']
        yield 'next', self.val['data']['next']

def lookup_type(val):
    if str(val.type) == 'task_t':
        return TaskPrinter(val)
    elif str(val.type) == 'Context':
        return ContextPrinter(val)
    return None
# Register class pretty-printer mapping
gdb.printing.register_pretty_printer(gdb.current_objfile(), lookup_type)

### 创建一个打印x变量的指令
class PrintVar(gdb.Command):

    def __init__(self):
        super(PrintVar, self).__init__("step_print", gdb.COMMAND_USER)

    def invoke(self, argument, from_tty):
        # Execute a single step
        gdb.execute("step")
        try:
            value = gdb.parse_and_eval("x")
            print(f"x: {value}")
        except gdb.error as e:
            print(f"error: {e}")
# Register the command with GDB
PrintVar()
# Invoke the command to print 'x'
gdb.execute("step_print")

### 创建一个执行一步打印寄存器值的回调
R = {}
def stop_handler(event):
    if isinstance(event, gdb.StopEvent):
        regs = [
            line for line in 
                gdb.execute('info registers',
                            to_string=True).
                            strip().split('\n')
                    if not line.startswith('xmm')
        ]
        for line in regs:
            parts = line.split()
            key = parts[0]
            if m := re.search(r'(\[.*?\])', line):
                val = m.group(1)
            else:
                val = parts[1]
            if key in R and R[key] != val:
                print(key, R[key], '->', val)
            R[key] = val
gdb.events.stop.connect(stop_handler)

### 通用的打印静态c-array
class ArrayPrinter:
    def __init__(self, val):
        self.val = val

    def to_string(self):
        array_type = self.val.type
        size = array_type.sizeof // array_type.target().sizeof
        # 打印数组内容
        result = "["
        for i in range(size):
            if i > 0:
                result += ",\n"
            result += str(self.val[i])
        result += "]"
        return result

def lookup_type(val):
    if val.type.code == gdb.TYPE_CODE_ARRAY:
        return ArrayPrinter(val)

### 指针解引用
val.dereference()
```

- gdb pretty-printer: [样例](https://zhuanlan.zhihu.com/p/492363382)  [Python-Scripting-API](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Python-API.html#Python-API) [Pretty-Printing-API](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Pretty-Printing-API.html#Pretty-Printing-API)

#### 常用
- info inferiors  # 打印进程/线程信息  (根据进程号 !cat /proc/{pid}/maps  # 打印进程的内存信息)
- starti  # 启动程序，并在第一条指令上暂停
- bt f    # backtrace full，打印堆栈信息
- b _start  # 在指定函数设置断点
- c  # continue 继续
- b *0x7c00 在指定地址断点
- target remote localhost:1234 # 远程qemu调试
- save breakpoints /path/to/breakpoints-file 保存断点信息
- finish: 退出当前堆栈
- p /x $r0: 打印寄存器值
- x $r0: 打印内存值(假设r0放着内存地址)
- 关闭输出分页: set pagination off (关闭--Type \<return\> to continue, or q)
- clear 指令用于删除特定位置的断点
- set $cpu = cpus[cpu_current()] : 建立表达式缩写
- x/i $pc: 查看指定地址的代码

##### 多线程调试常用指令
- info threads：查看线程状态信息
- thread <thread-id>：切换当前线程
- thread apply [thread-id-list | all] <command>：针对指定线程执行命令
	- 例如thread apply 2-3 bt：打印线程2和线程3的调用栈信息
	- taas command 相当于 thread apply all -s command
	- tfaas command 相当于 thread apply all -s -- frame apply all -s command
- break
	- break <location> 针对所有线程设置条件断点
	- break <location> thread <thread-id> if <condition>：设置条件断点，仅对指定线程生效
- set scheduler-locking mode(on/off/step/replay)
	- 控制是否锁定当前线程，如果当前程序被锁定，恢复程序运行时，只有当前线程可以运行
	- off：不锁定任何线程，当恢复程序执行时，所有线程都可以自由执行
	- on：锁定当前线程，执行continue、step、next、finish等命令时，只有当前线程恢复执行，其余线程仍然处于中断状态
	- step：当单步执行时，和on一样，其余情况和off一样。简单来说，就是在这个锁定模式下执行单步操作，只有当前线程会执行，其他线程仍然处于中断状态。而执行其他命令时，如continue、finish等，和off一样，即所有线程都可以自由执行。
	- replay(默认)：在反向调试时和on一样，其余情况下，和off一样
- **all-stop** vs **non-stop**模式
	- 默认处于All-Stop模式，即只要有一个线程被中断执行，其他所有的线程都会被中断执行。Non-Stop模式下，一个线程被中断执行，并不会影响到其他线程
	- set non-stop on：开启Non-Stop模式，该模式下，一个线程被中断执行，不会影响其他线程
	- set non-stop off：关闭Non-Stop模式，即进入All-Stop模式。该模式下，一个线程被中断执行，其他所有线程都会被中断
	- show non-stop: 查看Non-Stop模式是否开启
- interrut -a：中断所有线程执行
- command &：后台执行

#### emacs gdb
- .emacs配置
```lisp
(defun set-gdb-layout(&optional c-buffer)
  (if (not c-buffer)
      (setq c-buffer (window-buffer (selected-window)))) ;; save current buffer

  ;; from http://stackoverflow.com/q/39762833/846686
  (set-window-dedicated-p (selected-window) nil) ;; unset dedicate state if needed
  (switch-to-buffer gud-comint-buffer)
  (delete-other-windows) ;; clean all

  (let* (
         (w-source (selected-window)) ;; left top
         (w-gdb (split-window w-source nil 'right)) ;; right bottom
         (w-locals (split-window w-gdb nil 'above)) ;; right middle bottom
         (w-stack (split-window w-locals nil 'above)) ;; right middle top
         (w-breakpoints (split-window w-stack nil 'above)) ;; right top
         (w-io (split-window w-source (floor(* 0.9 (window-body-height)))
                             'below)) ;; left bottom
         )
    (set-window-buffer w-io (gdb-get-buffer-create 'gdb-inferior-io))
    (set-window-dedicated-p w-io t)
    (set-window-buffer w-breakpoints (gdb-get-buffer-create 'gdb-breakpoints-buffer))
    (set-window-dedicated-p w-breakpoints t)
    (set-window-buffer w-locals (gdb-get-buffer-create 'gdb-locals-buffer))
    (set-window-dedicated-p w-locals t)
    (set-window-buffer w-stack (gdb-get-buffer-create 'gdb-stack-buffer))
    (set-window-dedicated-p w-stack t)

    (set-window-buffer w-gdb gud-comint-buffer)

    (select-window w-source)
    (set-window-buffer w-source c-buffer)
    ))
(defadvice gdb (around args activate)
  "Change the way to gdb works."
  (setq global-config-editing (current-window-configuration)) ;; to restore: (set-window-configuration c-editing)
  (let (
        (c-buffer (window-buffer (selected-window))) ;; save current buffer
        )
    ad-do-it
    (set-gdb-layout c-buffer))
  )
(defadvice gdb-reset (around args activate)
  "Change the way to gdb exit."
  ad-do-it
  (set-window-configuration global-config-editing))


;; emacs gdb configuration
(global-set-key [M-left] 'windmove-left)
(global-set-key [M-right] 'windmove-right)
(global-set-key [M-up] 'windmove-up)
(global-set-key [M-down] 'windmove-down)

(global-set-key [f5] 'gud-run)
(global-set-key [S-f5] 'gud-cont)
(global-set-key [f6] 'gud-jump)
(global-set-key [S-f6] 'gud-print)
(global-set-key [f7] 'gud-step)
(global-set-key [f8] 'gud-next)
(global-set-key [S-f7] 'gud-stepi)
(global-set-key [S-f8] 'gud-nexti)
(global-set-key [f9] 'gud-break)
(global-set-key [S-f9] 'gud-remove)
(global-set-key [f10] 'gud-until)
(global-set-key [S-f10] 'gud-finish)

(global-set-key [f4] 'gud-up)
(global-set-key [S-f4] 'gud-down)

;; (setq gdb-many-windows t)
(setq gdb-many-windows nil)

;; enable mouse
(require 'xt-mouse)
(xterm-mouse-mode)
(require 'mouse)
(xterm-mouse-mode t)
(defun track-mouse (e))

(setq mouse-wheel-follow-mouse 't)

(defvar alternating-scroll-down-next t)
(defvar alternating-scroll-up-next t)

(defun alternating-scroll-down-line ()
  (interactive "@")
    (when alternating-scroll-down-next
      (scroll-down-line))
    (setq alternating-scroll-down-next (not alternating-scroll-down-next)))

(defun alternating-scroll-up-line ()
  (interactive "@")
    (when alternating-scroll-up-next
      (scroll-up-line))
    (setq alternating-scroll-up-next (not alternating-scroll-up-next)))

(global-set-key (kbd "<mouse-4>") 'alternating-scroll-down-line)
(global-set-key (kbd "<mouse-5>") 'alternating-scroll-up-line)


```
- 命令行emacs进入图形emacs模式, M-x键后输入gdb, 选择要调试的文件(需要gcc -g生成调试用符号)
- 使用GDB-Frames Disassembly打开额外的反汇编窗口

#### 进阶
- 众多前端：gdb-gui(推荐), cgdb(推荐), pwndbg, gdb-dashboard, vscode, ddd, ..
- Stack, optimized code, macros, ...
- Reverse execution 记录状态反向执行(通过target record-full记录)
- Record and replay( 类似Reverse execution)
- Scheduler 调度线程的执行

### 常用示例
- 设置动态链接库路径: set env LD_LIBRARY_PATH $LD_LIBRARY_PATH:.

#### 参考资料
- gdb实现原理(劫持被调试进程) https://zhuanlan.zhihu.com/p/336922639