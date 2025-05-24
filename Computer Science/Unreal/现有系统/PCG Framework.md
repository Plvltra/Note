- PCG Volume
	- 绑定PCG Component，控制撒点范围
	- 挂载PCGGraph，PCGGraph生成的内容会以BrushComponent的方式存在PCGComponent所在的Actor上
- PCG Graph
	- 以一种蓝图的形式表达PCG中程序化的过程，也就是资源生成的逻辑都是编写在该蓝图中
	- 支持子图嵌套, 将一个Graph作为另一个Graph的节点
- PCG Node
	- PCG Node目前大致分为三类, 撒点(Sampler节点)-点变换(Transform节点)-生成(Spawner节点)
	- 通过启动不同PCGNode Debug模式，查看Node的撒点结果
	- 自定义PCG Node: 新建蓝图继承于PCGBlueprintElement, 重写Fucntions的方法改变执行逻辑
- UPCGBlueprintElement: Epic官方留出来的专门用来扩展的基类
	- 目前可以扩展的地方
		- 基础属性
			- NodeTitleOverride：返回FName节点名字
			- NodeColorOverride：返回FLinearColor，节点颜色
			- NodeTypeOverride：返回EPCGSettingsType，可以是InputOutput，Spatial，Density等分类
		- 节点的执行方法
			- Execute:
			- ExecuteWithContext：相比于上一个方法会多传入一个FPCGContext类型的参数
		- 其他访问修改数据的方式
			- PointLoopBody 与 LoopOnPoints：遍历每个点，通过返回true/false决定是否输出结果，结果也是一个点。
			- MultiPointLoopBody 与 MultiLoopOnPoints：遍历每个点，返回一个数组。以达到可以返回多个点。
			- PointPairLoopBody 与 LoopOnPointPairs：传入两个点数组，遍历A并且遍历B，两两传入，通过返回true/false决定是否输出结果，结果是一个点。
			- IterationLoopBody 与 LoopNTimes： 传入两个SpatialData，循环N次，每次都执行LoopBody。

#### 代码分析
- UPCGComponent类
	- Generate(): 开始执行生成
- FPCGGraphExecutor: 调度器
	- UpdateGenerationNotification(): 更新任务完成数量的显示状态
- FPCGGraphTask: 执行的任务


#### 其他
- 参考资料
	- 通用介绍和入门示例: https://dev.epicgames.com/documentation/en-us/unreal-engine/procedural-content-generation-overview#attributeselector
	- 整体介绍: https://km.netease.com/v4/detail/blog/29017
	- https://www.youtube.com/watch?v=kF5bIhBj05Q