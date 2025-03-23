- gcc --verbose: 查看编译器使用的 include paths
- gcc -L 选项影响的是编译和链接阶段，它告诉 gcc 在编译和链接时去哪些目录查找库。
	LD_LIBRARY_PATH 影响的是运行阶段，它告诉操作系统在程序运行时去哪些目录查找动态链接库。