
由一些target构成
#### 函数
- 
#### 变量
- 使用变量名前加上 `$` 符号，最好用 `()` 或 `{}` 把变量给包括起来
```
objects = program.o foo.o utils.o
program : $(objects)
```
- 变量会在使用处类似C/C++中的宏展开
```
foo = c
prog.o : prog.$(foo)
    $(foo)$(foo) -$(foo) prog.$(foo)
```
- 变量可以用`=`赋值，右侧变量值可以定义在文件的任何一处
```
foo = $(bar)
bar = $(ugh)
ugh = Huh?
```
- 变量赋值`:=`只能使用前面已定义好了的变量
- 变量赋值`?=`如果未被定义过赋值，如果定义过这条语句将什么也不做

#### 环境变量
- 环境变量 `MAKECMDGOALS` ，这个变量中会存放你所指定的终极目标的列表
```
ifeq ($(MAKECMDGOALS),)
  MAKECMDGOALS  = image
  .DEFAULT_GOAL = image
endif
```
#### 隐含规则
1. 隐含规则: 把 `.o` 的目标的依赖文件置成 `.c` ，并使用C的编译命令 `cc –c $(CFLAGS)  foo.c` 来生成 `foo.o` 的目标
```
foo : foo.o bar.o
    cc –o foo foo.o bar.o $(CFLAGS) $(LDFLAGS)
隐式规则:
foo.o : foo.c
    cc –c foo.c $(CFLAGS)
bar.o : bar.c
    cc –c bar.c $(CFLAGS)
```


# CMake
#### 用例
``` cmake
# 打印verbose build信息
cmake  --build ./build -v

# 定义可配置的选项
option(ENABLE_DEBUG "Enable debug output" ON)

# 定义变量
set(SOURCE_FILES main.cpp)
# 引用变量
message("Source files: ${SOURCE_FILES}") 

# 相当于定义了一个函数或宏FUNCTION_NAME, 形参是arg1 arg2, 方便复用
function(FUNCTION_NAME arg1 arg2)
  # ...
endfunction()
macro(MACRO_NAME arg1 arg2)
  # ...
endmacro()

# 进入到子文件夹, 执行CMakeList.txt的内容
add_subdirectory(subproject)

# 根据文件创建静态/动态链接库
add_library(MyLib STATIC src/libfile1.cpp src/libfile2.cpp)

# 创建一个新的目标(有点像make的phony), 通常用于定义一些不生成文件的任务，比如清理工作、生成文档
add_custom_target
```

#### 资料
[入门语法](https://zhuanlan.zhihu.com/p/630144233)