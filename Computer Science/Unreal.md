- TMap和TSet的内部实现TSparseArray: https://zhuanlan.zhihu.com/p/364029291
- Unreal资源加载: https://zhuanlan.zhihu.com/p/357904199
- ECS架构: https://zhuanlan.zhihu.com/p/618971664
- UE5的ECS(MASS框架): 
	https://zhuanlan.zhihu.com/p/441773595  
	https://zhuanlan.zhihu.com/p/446937133  
	https://zhuanlan.zhihu.com/p/477803528
- Unreal GC: https://www.cnblogs.com/kekec/p/13045042.html
- UE5标识符详解: https://github.com/fjz13/UnrealSpecifiers
- UE反射机制: https://zhuanlan.zhihu.com/p/60622181
	- 反射C++代码.generated.h .gen.cpp 是由Unreal Build Tool和Unreal Header Tool产生。
		- UBT通过扫描头文件，记录所有包含反射类型的modules，当其中有头文件改变时，就会用UHT更新反射数据
		- UHT解析头文件，扫描标记，生成用于支持反射的C++代码
	- .generated.h
		- GENERATED_BODY()会生成一个由文件ID、行号、"GENERATED_BODY"字符组合成的宏, 例如: *FID_PanguForUnreal_Plugins_MetaToUI_Source_MetaToUI_Public_MetaAttributeSystem_AttrRoot_h_26_GENERATED_BODY*
		- generated.h文件底部会定义这个宏，就相当于我们的.h类定义中的GENERATED_BODY()会被宏展开
		- 上述宏由${MACRO}_RPC_WRAPPERS_NO_PURE_DECLS, ${MACRO}_INCLASS_NO_PURE_DECLS, ${MACRO}_ENHANCED_CONSTRUCTORS等几部分组成，分别是对UFunction的包装，构造的包装等
	- .gen.cpp
		- Z_Construct_UClass_${CLASS_NAME}: 生成本类的UClass对象
		- Z_Construct_UClass_${CLASS_NAME}_Statics: 保存本类各种元信息的结构体
		- StaticRegisterNatives${CLASS_NAME}: 向UClass对象注册Name->UFunction
	- UE NoExport对象(FVector等类没有UStruct()却能用UProperty修饰的原因)
		
		- NoExportTypes.h中写了所有这种类型的镜像类(有UStruct(noexport), UProperty等修饰)
		- NoExportTypes.h定义用!cpp 包裹, 猜测UHT不用cpp宏,生成这些类的反射代码(只生成Z_Construct_UClass_{CLASS_NAME},  Z_Construct_UClass_{CLASS_NAME}_Statics等静态内容，不生成注册代码)
		- 生成的反射文件是NoExportTypes.gen.cpp
		- 这些类由StaticClass, 有元信息，没有GeneratedBody展开和运行时注册UFunction逻辑。减少注册UFunction的开销。同时在一个文件里，UHT处理的更快
		- https://github.com/fjz13/UnrealSpecifiers/blob/main/Doc/zh/Specifier/UCLASS/UHT/NoExport.md
		- https://forums.unrealengine.com/t/why-are-there-two-definitions-of-uobject/511459/2


#### 调试技巧
- 调试内存踩踏: 使用Debug模式启动，在命令行Optional Argument里面加上 -StompMAlloc选项
- SlateUI布局规则: https://zhuanlan.zhihu.com/p/77681049
- 用户UI保存的位置:  Saved/Config/WindowsEditor/EditorPerProjectUserSettings.ini
- FPropertyValueImpl::GetObjectsToModify()根据FPropertyNode获取所在的UObject
- 通过watchpoint检查FString是否变化:  
	- **×** 对FString 所在地址头几字节打watchpoint无效, data的前几个字节不会变
	- **×** 对FString 当前wchar_t所在地址打watchpoint无效, FString赋值指向的地址可能会改变
	- **√** 对FString Data成员的ArrayNum或者Data的AllocatorInstance的Data指针值打断点

#### Python Debug
###### 1.在vs_code新增调试配置，如下所示，端口默认5678
```
{ 
	"configurations": [ 
		{ 
			"name": "UnrealEngine Python", 
			"type": "python", 
			"request": "attach", 
			"connect": { 
				"host": "localhost", 
				"port": 5678 
			}, 
			"redirectOutput": true 
		} 
	] 
}
```

###### 2.在UE编辑器控制台输入(逐行输入)
	
1. import debugpy_unreal
2. debugpy_unreal.install_debugpy()
	只需执行一次，如果安装失败，在命令行输入
	```
	import unreal
	print(unreal.get_interpreter_executable_path())
	```
	查看获取的路径，手动在python路径安装debugpy
	```
	pip install debugpy
	```
3. debugpy_unreal.start() //如果第一步配置的端口不是默认5678，则debugpy_unreal.start(your_port)
```
import debugpy_unreal
debugpy_unreal.install_debugpy()
debugpy_unreal.start()
```

###### 3、此时UE编辑器进入等待状态，在vs_code RunAndDebug启动进行调试
挂载成功的话UE恢复执行状态，可以在vs code中对python代码正常断点调试。


- unreal python启用stub提示: https://blog.csdn.net/Jingsongmaru/article/details/89493620, 然后再vscode settings.json中加上
```Python
"python.autoComplete.extraPaths": ["G:\\Project66\\Intermediate\\PythonStub"],
"python.analysis.extraPaths": ["G:\\Project66\\Intermediate\\PythonStub"]
```

#### 疑难杂症技巧
- UE Details面板滚动条问题:  是否展示滚动条是根据ActualSize/DesiredSize < 1出现滚动条，如果有滚动条问题，最好用WidgetReflector检查哪里的ActualSize异常，然后判断OnArrangeChildren计算的ActualSize正常。(这一次碰到的是SListPanel的OnArrangeChildren问题，WidgetReflector指向DetailsRow就能看到SListPanel ActualSize=Desired Size)
- UE Shader Cache存储路径: C:\Users\{UserName}\AppData\Local\UnrealEngine\Common\DerivedDataCache, 编译shader出问题可以考虑删掉

#### 小技巧
- 主要的回调在FEditorDelegates
- UnrealEditor.exe "PanguForUnreal.uproject" -UNATTENDED可以不点Send就自动发送Crash Report
- 可以通过遍历UClass GetDefaultObject获取一些Private类型的CDO

#### 配置
- uplugin 中添加"EnabledByDefault": true使工程加载插件