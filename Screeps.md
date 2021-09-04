# 基础篇



## 1、基础操作

### 产卵

`spawnCreep`方法

作用：该方法可以利用`spawn`（卵巢）来生成一个creep

格式：`Game.spawns['卵巢名'].spawnCreep( [组件], '名称' );`

### 选定

`Game.spawns['spawnName']`

可以通过这段代码选取spawn来指定这个卵巢生成creep

### 身体组件

生成的creep可以带有三个基础组件`[WORK,CARRY,MOVE]`和一个名称`Harvester`（命名这个creep）

==演示代码==

```js
//用卵巢生成一个爬虫

Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Harvester1' );
//选取名为Spawn1的卵巢，执行spawnCreep方法，生成一个叫做Harvester1的Creep，带有WORK,CARRY,MOVE组件
```



### 能源矿

![image-20210901175722209](C:\Users\mrain\AppData\Roaming\Typora\typora-user-images\image-20210901175722209.png)

​	能源矿开采的条件，必须带有WORK和CARRY组件，WORK组件来开采，CARRY组件来运输



### Script和Console

Console只要用于调试，只会执行一次代码

Script为脚本，可以循环执行，并且在离线后也可以进行



### 采集矿物

获取creep	`Game.creeps['creepName']`

寻找sources(能源矿)	`room.find(FIND_SOURCES)`	该方法是creep类下的方法，是creep可执行的指令

`creep.harvest(sources[0])`

移动爬虫	 `moveTo`	该方法时creep类下的方法

```js
//获取爬虫creep
	var creep = Game.creeps['Harvester1'];	
//让获取到的爬虫寻找能源矿的位置（找到的每个能源矿以数组的形式保存起来了）
	var sources = creep.room.find(FIND_SOURCES);
//如果找到的能源矿不在范围内，那么爬虫移动到矿的位置
    if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
        creep.moveTo(sources[0]);
    }
```



### 运输矿物

#### transfer方法

格式：`transfer(target, resourceType, [amount])`

target：运输目的地	例如：`Game.spawns['Spawn1']`

resourceType：能源类型	例如：RESOURCE_ENERGY	能源矿

amount：数量	采矿数量



#### 容量判断

获取爬虫的可用容量

格式：`getFreeCapacity([resource])`	该方法是creep类的方法

resource：能源类型	例如：RESOURCE_ENERGY	能源矿

```js
//获取爬虫
	var creep = Game.creeps['Harvester1'];
//如果爬虫的可用容量大于0，那么让爬虫采集矿物
    if(creep.store.getFreeCapacity() > 0) {
        var sources = creep.room.find(FIND_SOURCES);
        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
            creep.moveTo(sources[0]);06 
        }
    }else{		//如果没有可用的容量那么就开始往回运输，如果没有在target附近则前往移动
        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
            creep.moveTo(Game.spawns['Spawn1']);
        }
    }
```



### 生命周期

每个creep有1500游戏ticks(游戏的时间计量单位)寿命，过完1500ticks就会死亡



### for each遍历



#### in和of的区

```js
for each
for(var num in arr)	//得到的num是arr数组的索引值
for(var num of arr)	//得到的num是arr数组的值
```



#### 多项控制

```js
for(var num in Game.creeps){	//从Game.creeps的索引值循环
	var creep = Game.creeps[num];	//让每一次循环的creep为num下标的creep
    if(creep.store.getFreeCapacity() > 0) {
        var sources = creep.room.find(FIND_SOURCES);
        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
            creep.moveTo(sources[0]);
        }
    }else{		
        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
            creep.moveTo(Game.spawns['Spawn1']);
        }
    }
 }
```





## 2、自定义类

#### 基本格式

将自动采集运输编写到一个新的类

首先在左下角新建一个MODULES

```js
//把roleHarvester函数输出，可以在其他modules中使用（导出去）
module.exports = roleHarvester;
```



```js
//在main函数开头添加这句，你把他理解为C语言中的 #include
//这样就把这个函数导入到主函数中了，可以直接调用了
var roleHarvester = require('role.harvester');

roleHarvest.run();调用run函数
```

格式：`var roleHarvester = require('modules的文件名！')`

这个可以理解为java或者C#中的

```java
Date nowTime = new Date();
//同理
var a = require('Modules文件名')
```





#### ==示例==

```js
//自定义的类
var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
	}
};

module.exports = roleHarvester;
```



```js
//main函数
var roleHarvester = require('role.harvester');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        roleHarvester.run(creep);
    }
}
```



## 3、 战略性对象

==a key strategic object==

### Room Controller

RoomController	---	房间控制器

![image-20210904010947769](C:\Users\mrain\AppData\Roaming\Typora\typora-user-images\image-20210904010947769.png)

作用：通过控制这个结构（房间控制器），你可以在房间里建造设施，房间控制器的等级越高，你就可以建造越多和越高级的设施！



### 提升等级

提升控制器等级，需要一种新的工作型爬虫



```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Upgrader1' );
```

注意，这里创建了一个爬虫跟之前创建Harvester是没区别的，虽然名字叫做Upgrader但是仍然会执行之前的采矿任务，但是我们不希望它采矿，只希望它去提升控制器等级

因此我们用到了Memory（内存）



### Memory

Memory为内存,我们利用creep自身的内存属性，将自定义化的信息写入各个creep，这样不同的creep只会执行他们自己的代码。

所有存储的内存都可以通过全局 Memory 对象访问



```js
Game.creeps['Harvester1'].memory.role = 'harvester';
Game.creeps['Upgrader1'].memory.role = 'upgrader';
//给指定名称的creep的memory的role写入指定函数
//通过这两个就能把他们的内存信息分配，之后可以配合if判断来特定选取
```



在这里我们定义一个upgrader函数

```js
var roleUpgrader = {

    run: function(creep) {
        //如果creep的储存量为0，那么去寻找矿物并采集
	    if(creep.store[RESOURCE_ENERGY] == 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }	//否则去寻找controller，去升级upgradeController
        else {
            if(creep.upgradeController(creep.room.controller) == ERR_NOT_IN_RANGE) {
                creep.moveTo(creep.room.controller);
            }
        }
	}
};

module.exports = roleUpgrader;
```





### 判断区分



```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {
    //for in 循环，循环room里的所有creeps
    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        //如果这个creep的memory的role为收割者，那么运行roleHarvester函数
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        //如果这个creep的memory的role为升级者，那么运行roleUpgrader函数
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
    }
}
```



## 4、基础结构

### 能源拓展组件

**Extensions**拓展组件相当于额外电池，帮你储存更多的能量，建造需要更多能量的爬虫

因此提供了建造更加强大的爬虫的能力，给予一个爬虫更多组件会让它获得更高的效率，初始的卵巢只能建造最高300能量的爬虫，拓展组件可以放在room的任意位置



### 建造EXTENSION结构

#### 生成带有memory的creep

```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Builder1',
    { memory: { role: 'builder' } } );
//用{memory:{role:'builder'}} 等价于 Game.creeps['Builder1'].memory.role = 'builder';
//{memory:{role:'builder'}}是一种	JSON字符串 表示法
```

如果你想了解JSON字符串表示法 请点击

https://www.cnblogs.com/abella/p/11125685.html

由此我们引出，spawnCreep的完整用法



`spwanCreep(body,name,[opts])`

body：身体组件	例如[WORK,CARRY,MOVE]

name：名字

[opts]：选择	利用JSON格式，最佳例子是给其分配memory



#### Builder建造者

`building`属性

creep.memory.building是一个布尔值类型数据

`visualizePathStyle`可视化路径样式

让路径可视化



由此我们引出moveTo的完整用法

moveTo(x,y,[opts])

moveTo(target,[opts])

x,y：坐标值

target：目标点

[opts]：其中一个例子便是visualizePathStyle路径可视化，同样使用JSON格式

例如

`{`
    `fill: 'transparent',`
    `stroke: '#fff',`
    `lineStyle: 'dashed',`
    `strokeWidth: .15,`
    `opacity: .1`
`}`

```js
role.Builder
var roleBuilder = {
    
    run: function(creep) {
		//如果building为true 并且 爬虫的能源矿储存量为0，那么就把building改为false，并且爬虫发出🔄 harvest
	    if(creep.memory.building && creep.store[RESOURCE_ENERGY] == 0) {
            creep.memory.building = false;
            creep.say('🔄 harvest');
	    }
        //如果building为false 并且 爬虫剩余储存空间为0，那么就把building改为true，并且爬虫发出🚧 build
	    if(!creep.memory.building && creep.store.getFreeCapacity() == 0) {
	        creep.memory.building = true;
	        creep.say('🚧 build');
	    }
		
        //如果爬虫的builing为true，那么寻找房间内的建筑点
	    if(creep.memory.building) {
	        var targets = creep.room.find(FIND_CONSTRUCTION_SITES);
            //如果建筑点不为空，进行下一判断
            if(targets.length) {
                //建造targets[0] 如果不在范围内则向targets[0]移动
                if(creep.build(targets[0]) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
	    }
	    else {
	        var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
	    }
	}
};

module.exports = roleBuilder;
```



**然后利用判断区分来操控Builder建造者**

![image-20210904235953933](C:\Users\mrain\AppData\Roaming\Typora\typora-user-images\image-20210904235953933.png)

建造者建造的最初形式为EXTENSION







### 寻找结构

`Game.structures`	获取所有的结构

或者

`Room.find(FIND_STRUCTURES)`		寻找房间里的结构

但是以上方法均为找到结构类型的东西，但我们还需要进一步筛选所以可以利用structure的structureType配合判断来筛选

`structure.structureType == STRUCTURE_EXTENSION`

`structure instanceof StructureExtension`

这两种都可以，同理



### 更聪明的Harvester

我们一开始的Harvester只会将能源运到卵巢，但是结构同样也需要能源，所以我们利用structure的判断改变代码，让Harvester变得更聪明

```js
var roleHarvester = {
    run: function(creep) {
        //前面开采没有任何改变
	    if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
        }
        else {
            //这里寻找结构，使用lamda函数，返回的是符合 (结构为EXTENSION或者为SPAWN) 且 结构的能源存储大于0的 将能源运达
            var targets = creep.room.find(FIND_STRUCTURES, {
                    filter: (structure) => {
                        return (structure.structureType == STRUCTURE_EXTENSION || structure.structureType == STRUCTURE_SPAWN) &&
                            structure.store.getFreeCapacity(RESOURCE_ENERGY) > 0;
                    }
            });
            if(targets.length > 0) {
                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
	}
};

module.exports = roleHarvester;
```



由此我们引出find的完整用法

find(type,[opts])



### 能源概况

`energyAvailbale`

room底下的数据，用来查看房间内所有生成和扩展的可用能量总量。

`console.log(str)`控制台输出



### 墙

### 城墙
