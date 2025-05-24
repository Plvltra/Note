- launch.json: 启用断点调试
	- 希望以项目某个Module作为入口，例: "module": "common.server.micro_server"
	- 希望调试单个文件: "program": "main.py",
- 好用项目管理的插件: Project Manager
- 开启多行Tab: Open Workspace Settings -> workbench.editor.wrapTabs
- 在vscode远程linux服务器
	1. 进入Remote Explorer插件, 使用Remotes(Tunnels/SSH)
	2. 在SSH的设置中在.ssh/config新建一个配置如下示例，或者用+号创建一个
		```
		Host PanguMicroService
			HostName 10.95.37.67
			User zhangzhengxun
			Port 32200
			IdentityFile ~/.ssh/id-rsa
		```
	3. 建立连接
- sftp上传代码到远程服务器
	1. 安装sftp插件
	2. 本地项目配置sftp.json，示例
		```
		{
		    "name": "zhangzhengxun",
		    "host": "10.95.37.67",
		    "protocol": "sftp",
		    "port": 32200,
		    "username": "zhangzhengxun",
		    "privateKeyPath": "C:/Users/zhangzhengxun/.ssh/id-rsa",
		    "remotePath": "/home/zhangzhengxun/MSPangu",
		    "uploadOnSave": true,
		    "ignore": [".vscode", ".git", ".DS_Store"],
		    "localPath": "."
		}
		```
	3. sftp插件upload
- 配置clang-format: https://skillupwards.com/blog/formatting-c-cplusplus-code-using-clangformat-and-vscode
- 配置文件
	- tasks.json: 
		- 在”Terminal“菜单下点击”Configure Tasks...”子菜单。在随后出现的弹出框中选择
		- 用来是设置指令编译代码。
	```json
		{
	    // 配置make任务
	    "version": "2.0.0",
	    "tasks": [
	        {
	            "label": "make",
	            "command": "make -B",
	            "type": "shell",
	            "options": {
	                "cwd": "${fileDirname}" // 当前文件所在的目录
	            }
	        }
		}
	```

	- launch.json: 
		- 点击“Run”菜单下的“Start Debugging”子菜单, 然后配置launch.json
		- 用来设置执行环境来执行代码。（如果不设置每次点击调试的时候都需要手动选择debugger）
	
	```json
		// launch.json调试c
		{
		    "version": "0.2.0",
		    "configurations": [
		        {
		          "name": "C/C++: gcc build and debug active file",
		          "type": "cppdbg",
		          "request": "launch",
		          "program": "${fileDirname}/crepl-64", // 可执行文件路径
		          "args": [],  // 传递给程序的命令行参数
		          "stopAtEntry": false,
		          "cwd": "${fileDirname}", // 工作目录
		          "environment": [],
		          "externalConsole": false,
		          "MIMode": "gdb",
		          "setupCommands": [
		            {
		              "description": "Enable pretty-printing for gdb",
		              "text": "-enable-pretty-printing",
		              "ignoreFailures": true
		            }
		          ],
		          "preLaunchTask": "make", // 编译任务
		          "miDebuggerPath": "/usr/bin/gdb", // gdb 调试器路径
		          "logging": {
		            "engineLogging": true
		          }
		        }
		    ]
		}

		// launch.json使用sudo调试c/c++
		{
		    "miDebuggerPath": "${workspaceFolder}/gdb_root.sh"
		}
		// gdb_root.sh
		#!/bin/bash
		SELF_PATH=$(realpath -s "$0")
		if [[ "$SUDO_ASKPASS" = "$SELF_PATH" ]]; then
			zenity --password --title="$1"
		else
			exec env SUDO_ASKPASS="$SELF_PATH" sudo -A /usr/bin/gdb $@
		fi

		// launch.json调试python
		{
		    "version": "0.2.0",
		    "configurations": [
		        {
		            "name": "ld - hello",
		            "type": "debugpy",
		            "request": "launch",
		            "program": "${workspaceFolder}/fle/demo/ld",
		            "console": "integratedTerminal",
		            "args": ["foo.fle", "main.fle", "libc.fle", "-o", "hello"],
		            "cwd": "${workspaceFolder}/fle/demo",
		        },
		    ]
		}
	```
	
	- c_cpp_properties.json: 
		- 可以通过设置"defines"控制vscode宏打开情况, "includePath"跳转可搜集的路径
	


- vscode调试qemu
```json
{
	"version": "0.2.0",
    "configurations": [
        {
            "name": "Debug with GDB Script",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/kernel-x86_64-qemu.elf",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerServerAddress": "localhost:1234",
            "miDebuggerPath": "/usr/local/bin/gdb",
            "setupCommands": [
            ],
            "logging": {
                "trace": true,
                "traceResponse": true,
                "engineLogging": true,
                "programOutput": true,
                "exceptions": true
                
            }
        }
    ]
}
```

#### 小技巧
- [Vscode写C++环境配置](https://www.zhihu.com/question/353722203)
- 在vscode中调用gdb命令: 以-exec开头, 接自己的命令
	```
	-exec set follow-fork-mode child # 调试子进程
	```
- Move Terminal into Editor Area : 可以设置Terminal以类似文件的展示(Move Terminal into Panel切回下方面板)

#### Vscode Python自动补全及语法检查

首先需要安装Visual Studio IntelliCode扩展程序  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f868aaa6158bcac4ee1589c6iNyeipN01?sign=2fn8HEUIo6_AY5WSa61zqKdDHNI=&expire=1732673955)  
接下来需要进行一些项目设置，打开“文件-首选项-设置”，搜索“python.languageServer”，选择你想使用的python语言服务器。需要注意的是，在搜索框下面可以选择用户设置/工作区设置，工作区是仅限于当前工程目录，保存在.vscode/setting.json中的，而用户则是对于所有项目的。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f868c336158bcbd1bf863a5wvF7TVW201?sign=-VQzJr1NHLbw_Qf77_ExlcXnTfI=&expire=1732673955)  
Visual Studio IntelliCode建议使用Microsoft语言服务器。也可以使用默认的Jedi，设置python.jediMemoryLimit可以为其分配更多的内存以提供更精确的类型判断。设置为None的话也有关键词补全，但没有类型推断函数补全。Pylance是一个集合了许多Python IDE功能的扩展程序，基本上只要装上就环境补全语法检查什么都有了，并且作为语言服务器功能更强大。然鹅Pylance是针对Python3的，NeoX的2.7用不了，嘤嘤嘤。  
除了language server以外，还需要设置一下补全库。由于NeoX可能不包含python库的所有内容，这里还是先将项目环境设置到NeoX的python环境下。在设置中搜索“python.pythonPath”，指定路径。也可以在窗口最下面状态栏的左边点击设置。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f868f8c2dcade7ab4054ba9sfcbcjtJ01?sign=dPD1jsrdkCP81yY2NfGLEdzsAPc=&expire=1732673955)  
设置好工程环境之后，发现VS Code对于neoX中的包并不能识别，不仅无法自动补全，还会有小黄线警告。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f8690586158bcbd1bf8654abcwj5FNb01?sign=7cTUwL-Xe_12FNmgHCjthlDpnXU=&expire=1732673955)  
这是由于NeoX环境下的许多包都是DLL，语言服务器无法识别python接口。为解决这个问题，我们需要给语言服务器添加额外路径。在设置中搜索“python.autoComplete.extraPaths”，选择“在setting.json中编辑”，将我们项目sys.path中的路径添加进去。然后添加NeoX接口的python定义路径，可以在引擎目录下的pystub文件夹中找到。设置完成后大概是下面这样子：

```swift
    "python.autoComplete.extraPaths": [
        ".\\script",
        "C:\\Users\\shenbaozhen\\AppData\\Local\\neox-hub\\NeoX-Engine_2020.07.20s1\\Core-Editor\\win32_editor\\pystub"
    ]
```

重启VS Code，此时发现由于找不到模块名称的小黄线都消失了，自动补全也可以使用了。但是由于语言服务器是静态检查，对于一些动态变量仍然无法识别，还是会有小黄线并且无法补全。现在正在寻找进一步的解决方案，已知可以使用snippets进行变量补齐，在C:\Users\xxxxxxx\AppData\Roaming\Code\User\snippets目录添加python.json文件进行定义，详见[Snippets in Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets)  
对于有些项目没有使用标准的NeoX库，没有pystub目录之类的情况，可以使用NeoX的文档生成工具，详见NeoX Wiki：[NeoX API生成工具](http://neox.netease.com/docshow/#/92.Related%20Tools/02.%E7%BC%96%E7%A8%8B%E7%9B%B8%E5%85%B3%E5%B7%A5%E5%85%B7/02.NeoX%20API%E7%94%9F%E6%88%90%E5%B7%A5%E5%85%B7/)

---

##### 语法检查&自动格式化

python扩展程序提供了linting的基本功能与框架，除了默认的pylint以外也可以选择bandit，flake8，mypy等语法静态语法检查工具，它们都是基于python的，可以用pip安装。pylance的pyright则是不基于python的，需要安装扩展程序。我使用的linting是flake8，除了语法检查外还有pep8规范的检查，对养成良好的编程习惯很有帮助，虽然报错和警告会非常多就是了。其他的linting配置可以在VS Code文档上查看[Linting Python in Visual Studio Code](https://code.visualstudio.com/docs/python/linting)

##### flake8配置

首先需要启用flake8，在设置中搜索关键词“python linting enabled”，选择启用linting和启用flake8，其他linting禁用即可。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f869e2868d8643002e6ce16DavlzYMK01?sign=F_4I7S-WOGxMWlWd_3b7xIhFiXg=&expire=1732673955)  
假如没有安装flake8，VS Code会提示安装。由于NeoX的python不含pip并且不使用site共享库，因此需要我们手动在控制台`pip install flake8`。等待安装完成之后，手动使用系统pip安装的话，需要指定外部的flake8路径。在设置中搜索关键词“python.linting.flake8”，将flake8的路径设置为这个路径，具体位置可能因人而异。注意是flake8.exe的路径，而非其python源文件目录。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f869f9b6158bcb18683a1eawqiq9aIB01?sign=CaVC-F4PUyb4ynB_uh17uqyijRM=&expire=1732673955)  
设置好路径之后，随便打开一个.py文件，就会发现代码被红色波浪线和黄色波浪线淹没了。这是因为NeoX并不是按照pep8规范进行编写的。因此我们需要设置flake8的参数，对一些pep8规则进行过滤。在刚才的设置搜索内容下，找到Flake8 Args，点击“添加项"，输入`--ignore=E301, E306, W`。其中”E301"表示E类错误中的E301这一种类型的报错，W表示所有W类型的报错。这些报错的具体编号可以将鼠标悬停到波浪线上方查看。也可以在flake8的文档中查询[Configuring Flake8](https://flake8.pycqa.org/en/latest/user/configuration.html)  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f86a5f268d8641fd12546a3DtZXDgIQ01?sign=ZI5704EUL-V9yY8hxoHj3xpyc8o=&expire=1732673955)  
编辑启动参数在json里直接写会比较方便，可以打开.vscode/setting.json编辑工作区设置。除了“--ignore”参数以外，flake8还有许多启动参数可以设置，在命令行输入`flake8 --help`可以查看。譬如可以设置`--max-line-length=360`，表示允许一行最大长度，默认为79。  
在设置中还可以更改flake8每种类型错误的提示波浪线类型，我比较喜欢全部都设为Information小蓝线，比较容易注意到又不会太刺眼。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f86a5a268d8643002e6cf9chVpvbHsh01?sign=E6wsuSMXy602nJr4dzK_J9u5jng=&expire=1732673955)  
设置完成后，每次保存代码就会刷新对代码的语法检查，为了看语法检查，自然而然会养成时刻Ctrl+S的好习惯。

### yapf配置

在flake8及其严格的监管之下，划线实在太多了，手动改比较费时间，有时候也是需要借助自动化工具的。vscode自动化排版工具有autopep8，black，yapf，快捷键Shift+Alt+F。它们同样是基于python的，可pip安装，NeoX需要设置外部路径，这里不再详述。我使用的是yapf，没什么特别的理由。设置中搜索关键词“python formatting yapf”，选择Provider设好路径，准备添加参数。  
![image.png](https://kmpvt.pfp.ps.netease.com/file/5f86ac4c68d8643002e6d0f7rYKo8BXx01?sign=e-zpxp9rDYzs8i_9UWyqqjjVim0=&expire=1732673955)  
yapf的参数设置主要只设置“--style"，可以用`yapf --help`查看所有参数帮助，或`yapf --style-help`查看style的详细参数帮助。这里只提两个比较关键的style参数。因为NeoX引擎代码里大部分都是用Tab键缩进，而默认情况下yapf会使用4个空格进行对其，因此需要设置`use_tabs: true`；当一行长度超过一定字符数之后，yapf会对行进行拆分，由于Tab缩进的关系，需要设置`continuation_align_style=FIXED`或将行长度阈值调大`column_limit: 100000`

所有内容设置完成之后，工作区的setting.json文件大致如下：

```swift
{
    "python.languageServer": "Microsoft",
    "python.pythonPath": "c:\\Users\\shenbaozhen\\AppData\\Local\\neox-hub\\NeoX-Engine_2020.07.20s1\\Core-Editor\\win32_editor\\python.exe",
    "python.autoComplete.extraPaths": [
        ".\\script",
        "C:\\Users\\shenbaozhen\\AppData\\Local\\neox-hub\\NeoX-Engine_2020.07.20s1\\Core-Editor\\win32_editor\\pystub"
    ],
    "python.linting.pylintEnabled": false,
    "python.linting.flake8Enabled": true,
    "python.linting.mypyEnabled": false,
    "python.linting.flake8Path": "C:\\Users\\shenbaozhen\\AppData\\Roaming\\Python\\Scripts\\flake8",
    "python.linting.flake8Args": [
        "--ignore=E301, E306, F821, W191, E221, W293, E128, E256",
        "--max-line-length=360"
    ],
    "python.formatting.yapfPath": "C:\\Users\\shenbaozhen\\AppData\\Roaming\\Python\\Scripts\\yapf",
    "python.formatting.yapfArgs": [
        "--style={based_on_style: pep8, use_tabs: true, indent_width: 2, column_limit: 360, continuation_align_style=FIXED, spaces_before_comment=2}"
    ],
}
```

---

```armasm
Visual Studio Code还有许多自定义设置，丰富的插件帮助开发，这些就留给大家自己去慢慢发现，寻找适合自己的配置方式。本篇还残留着些许配置不当的情况，我会在后续使用中慢慢改进，更新到这篇文章上。
```

更多关于VS Code开发python的内容可以参考VS Code官方文档说明[Editing Python in Visual Studio Code](https://code.visualstudio.com/docs/python/editing)