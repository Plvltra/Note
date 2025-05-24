
#### Snap Grid Flow Builder
概念: 
`Module`: A room is designed in a separate map file, and it is called a `Module`

`Connection`
- A `Connection` is a stitching point that the Snap framework uses to join rooms together
- Snap Connection用于摆放在关卡的四周蓝标处, 用于连接其他关卡

`snap room modules:`  
required to be of a fixed size
User can design snap rooms to span multiple nodes in the flow graph

 `goal room:`
 the goal room was designed to be 2x2x2 the size of the chunk

`snap modules`
The flow framework will also read the available doors you've setup in your snap modules and use that configuration to grow the graph。

`flow graph`资产
- Create Main Path的Module Categories可以关联到 Module DB的Modules Array的Category
- Create Main Path: 可以build出主干节点
- Create Path: 可以build出分叉节点（Start from Path, End on Path） 
- 可以在flow graph的Create Path节点设置Module Category Override Method 
	- 例如: CreateMainPath节点设置Module Category Override Method为Start / End Nodes，并且End Node Category Override新加一个Boss, 那么main graph的最后一个节点就是Boss房间
- SpawnItems 节点作用: 在Paths所在的路径节点上(例如main路径, 分支路径)生成marker对应的对象

`APikeLayerInfo`类型Actor
- 内容是引用不同的关卡，以实现在一个Level中的不同Level的切换(好像是Build Dungeon生成的)
- `Layers Data`: 

`Module Database`资产
- 内容是引用不同的关卡，以实现在一个Level中的不同Level的切换(好像是Build Dungeon生成的)
- Modules Array的All Layers字段: 似乎是Level所有LayerInfo的Layer Map收集?
- Modules Array的Layers Data字段: 似乎是Level所有LayerInfo的LayerData收集

`Snap Grid - Module Bounds`资产
- Dungeon - Num Chunks属性: 表示当前关卡所占的格子大小

`Dungeon` Actor对象
-  Dungeon - Builder Class属性是实际的Build类型(例如SnapGridFlowBuilder)
- 需要指定`Module Database`, `Flow Graph` and `Theme`对象

`Placeable Markers`
- 创建Placeable Markers Asset，并摆放到Tile里面
- 在Theme里面Add Marker Node, 连接上Mesh，就是Placeable Markers的内容
- Build Dungeon之后，Level摆放Placeable Markers Asset的地方就会Spawn生成的


## Pike Dungeon
- `AllLayers`: are all the available sublevels found in the tile.
- `LayersData`: are collected getting all `APikeLayerInfo` actors present in the tile and its sublevels. They actually represent the real tile variants data and are set by level designers.
