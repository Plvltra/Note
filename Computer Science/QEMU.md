- 如果希望在启动后进入 QEMU Monitor，可以使用 `Ctrl + A` 然后按 `C` 来切换到 QEMU Monitor 界面。如果你在使用 `-nographic` 模式下运行 QEMU，这个组合键进入 Monitor。
- gdb 远程连接qemu -serial  stdio 从串口输出的内容似乎没有经过stty处理的配置
	``` cpp
	#define COM1 0x3f8
	outb(COM1, ch);
	```
	一个表现错误的例子是wsl \r\n换行并移动光标到行首。即使stty开启了onlcr(translate newline to carriage return-newline), 输出的\n也没有变成\r\n