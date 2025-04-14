#### 用例
``` cmake
######## 整体使用 ########
cmake -B . -S build # 指定源文件路径, 在build下生成目标构建工具构建文件

# 打印verbose build信息
cmake  --build ./build -v

# 指定TARGETS和.h的安装目标位置
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
# 指定包体安装位置
cmake --install . --prefix "/home/myuser/installdir"

# 添加新的test, 指定对应的命令, 指定预期结果
add_test(NAME Runs COMMAND Tutorial 25)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5"
  )
# 使用ctest
ctest -N # 打印全部test列表
ctest # 执行全部ctest
ctest -VV # 执行全部ctest并打印详细信息

######## 语法 ########
# 定义可配置的选项
option(ENABLE_DEBUG "Enable debug output" ON)

# 定义变量
set(SOURCE_FILES main.cpp)
# 引用变量
message("Source files: ${SOURCE_FILES}") 

# 相当于定义了一个函数或宏FUNCTION_NAME, 形参是arg1 arg2, 方便复用
function(FUNCTION_NAME arg1 arg2)
  ...
endfunction()
macro(MACRO_NAME arg1 arg2)
  ...
endmacro()

# 进入到子文件夹, 执行CMakeList.txt的内容
add_subdirectory(subproject)

# 根据文件创建静态/动态链接库
add_library(MyLib STATIC src/libfile1.cpp src/libfile2.cpp)

# 创建一个新的目标(有点像make的phony), 通常用于定义一些不生成文件的任务，比如清理工作、生成文档
add_custom_target

# 添加项目的include路径
include_directories("${PROJECT_SOURCE_DIR}/util")

######## Generator Expressions ########
`$<0:...>` 返回空, and `<1:...>` 返回 `...`

# 根据语言是否是CXX和编译器类型返回1或0
$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>

# BUILD_INTERFACE在Build阶段是1, 控制build阶段满足编译器开启选项
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)

######## 打包 ########
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
# 生成包体
cpack -G ZIP -C Debug
# 生成一个包含源码的打包文件
cpack --config CPackSourceConfig.cmake

######## 内置变量/函数说明 ########
CMAKE_CURRENT_SOURCE_DIR: 表示当前CmakeLists.txt所在的文件目录
```


#### 资料
- [官方Matering CMake系列](https://cmake.org/cmake/help/book/mastering-cmake/index.html)
	- [tutorial](https://cmake.org/cmake/help/book/mastering-cmake/cmake/Help/guide/tutorial/index.html)
- [官方文档细节根目录](https://cmake.org/cmake/help/latest/index.html)
	- [内置Command](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)
	- [内置Variable](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html)
- 其他文档
	[入门语法](https://zhuanlan.zhihu.com/p/630144233)

#### 概念
- CMakeCache.txt: CMake使用这个文件记录一堆the user’s selections and choices, 可以通过ccmake或者直接修改CMakeCache文本修改选项
-  Generator expressions: 在build system generation执行时根据具体的build configuration(例如编译期类型)求值

#### 细节
- 使用引号包裹路径可以使得路径被正常解析: i.e. target_include_directories(Tutorial PUBLIC "${PROJECT_SOURCE_DIR}"), 没有引号如果路径包含空格或特殊字符可能会导致路径处理错误
- 查看CMakeLists.txt的变量展开: cmake --trace-expand build


# Make
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
