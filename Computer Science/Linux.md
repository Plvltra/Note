#### 记录
- shell编程和工具入门资料: https://missing.csail.mit.edu/2020/shell-tools/
- shell数据整理(转换): https://missing.csail.mit.edu/2020/data-wrangling/
- perf获取火焰图
``` sh
perf record -e cpu-clock -g xxx # 记录一段程序
perf script -i perf.data &> perf.unfold
FlameGraph/stackcollapse-perf.pl perf.unfold &> perf.folded
FlameGraph/flamegraph.pl perf.folded > perf.svg
``` 
-  主要目录用处
	- /usr/bin: 系统二进制 
	- $HOME/bin: 个人二进制(不用sudo可能会安装到个人二进制)
	- /usr/lib: 系统链接库 
	- /usr/local: 通常用于存放不由包管理，用户手动安装的软件和库
- apropos: 查看工具对应的文档
- sudo -i: 进入root user模式

#### 约定习惯
- /etc/alternatives/: 通常用于管理多版本软件, 如python3有多个版本, 指定实际指向的程序路径符号链接

#### 工具
- `lstopo`: 获取cpu拓扑排序图
- `stty`: stty是一个用于配置和显示终端（terminal）设置的 Unix/Linux 命令。可以终端传输波特率 、控制终端是否开启onlcr(translate newline to carriage return-newline)等行为
- history: - 键入 `history` 查看命令行历史记录，再用 `!n`（`n` 是命令编号）就可以再次执行。其中有许多缩写，最有用的大概就是 `!$`， 它用于指代上次键入的参数，而 `!!` 可以指代上次键入的命令了（参考 man 页面中的“HISTORY EXPANSION”）。不过这些功能，你也可以通过快捷键 **ctrl-r** 和 **alt-.** 来实现。
- 回到前一个工作路径：`cd -`
- `top` 命令用于实时显示系统性能信息
- "--" in commands :  to signify the end of command options，例如**grep -- -v file**，此处的-v不会被认为是选项
- ctrl+c: send SIGINT to terminate a process
	ctrl+z: put a process to background
	ctrl+d: send EOF to stdin
- ./program > /dev/null 丢弃所有标准输出，只打印标准错误
- ls \*\*/\* 递归打印目录 (globbing技巧) 
- unlink: 减少文件的软链接引用数量
- 

#### 配置(杂)
- wsl2装perf: https://blog.csdn.net/arong_xu/article/details/136965827 (似乎还是有编译问题)
- 使用源码装可配置的musl-libc: 
	```
	./configure --enable-debug --exec-prefix=/usr(指定)
	make && sudo make install	
	```
- **使用dlpkg可以列出并删除难删的Package** (dpkg -l | grep -E 'qemu|riscv' | awk '{print $2}' | xargs sudo apt remove --purge -y) 删除qemu
- risc-v指令集架构依赖工具: gcc-riscv64-linux-gnu(riscv64-unknown-elf-gcc)  binutils-riscv64-linux-gnu

#### API
##### select与epoll (两种同步io)
**背景**: 服务端需要管理多个客户端连接，而recv只能监视单个socket。epoll的要义是高效的监视多个socket。从历史发展角度看，必然先出现一种不太高效的方法，人们再加以改进。
- select的用法: 
	- 准备一个数组fds表示所有需要监视的socket，然后调用select。如果fds中的所有socket都没有数据，服务端进程会阻塞。直到有一个socket接收到数据，select返回，服务端进程被唤醒。进程可以遍历fds，通过FD_ISSET判断socket是否收到数据，然后做出处理。
- 缺点
	- 每次调用select都需要将进程加入到所有监视socket的等待队列，每次唤醒都需要从每个队列中移除,涉及了两次遍历。而且每次将整个fds列表传递给内核也有一定开销。
	- 服务端进程被唤醒后，不知道哪些socket收到数据，还需要遍历一次。
- epoll介绍
	- 设计思路
		- 功能分离: 拆解select功能, 使用epoll_ctl维护等待队列，再调用epoll_wait阻塞进程
		- 就绪列表: 内核维护一个rdlist(就绪列表)。 进程被唤醒后，只要获取rdlist的内容，就能够知道哪些socket收到数据
	-实现分析
		- epoll_create会让内核创建一个eventpoll对象, epoll_ctl会将eventpoll放入对应socket等待队列，epoll_wait会让内核将进程A放入eventpoll的等待队列。socket收到数据后，中断程序会在eventpoll的rdlist添加socket引用，唤醒eventpoll，使程序再次运行。

- 参考资料: epoll(https://zhuanlan.zhihu.com/p/63179839 https://zhuanlan.zhihu.com/p/64138532 https://zhuanlan.zhihu.com/p/64746509)

#### 便捷工具
- Oh My Zsh: https://zhuanlan.zhihu.com/p/35283688
	- 推荐插件: git, vscode,  zsh-autosuggestions, zsh-syntax-highlighting，z
	- 推荐字体: powerlevel10k
	- 插件和字体全都存在~/.oh-my-zsh/custom, 配置在~/.zshrc
- 软件tldr, most, tree
 
#### 电脑迁移
``` sh
# 源电脑
dpkg --get-selections > packages.txt #保存安装包
cp -r ~/{.bashrc,.zshrc,.vimrc,.gitconfig,.oh-my-zsh} {tempdir} #备份配置

# 目标电脑
sudo apt-get update
sudo apt-get install dselect
sudo dpkg --set-selections < /mnt/c/temp_configs/packages.txt #安装安装包
sudo apt-get dselect-upgrade

cp -r {tempdir} {.bashrc,.zshrc,.vimrc,.gitconfig,.oh-my-zsh} ~/ #恢复配置
```