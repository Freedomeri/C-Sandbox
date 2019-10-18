#第二十四节 C++搭建行为树框架、实现巡逻AI  

##24.1C++搭建行为树框架  

在学习C++行为树之前，建议先从网上自学一下相关知识。  



1、要使用AI行为树，首先在Build.cs文件里面引入两个模块AIModule和GameplayTasks：  

![](https://i.imgur.com/JKNoWTb.png)  

然后从VS运行一下虚幻编辑器，以加载AI相关的模组。  

2、打开虚幻编辑器，在Content/Blueprint/Enemy文件夹下新建一个行为树：  

![](https://i.imgur.com/Sft2Uxf.png)  

命名为EnemyBehaviorTree：  

![](https://i.imgur.com/4ePmLsY.png)  

3、开始创建C++文件。  
在Public文件夹下新建AI文件夹，并在里面新建C++类，点击ShowAllClasses，选择继承BlackBoardData类：  

![](https://i.imgur.com/iISRSKS.png)  
我们可以把黑板当作是内存，行为树当作CPU，CPU需要的变量会放到内存中。之后写C++用这种思想比较好理解。   



再新建一个类，继承BTDecorator类，是行为树的一个判断器：  

![](https://i.imgur.com/hWwRSwY.png)  

继续创建第三个C++类，继承BTService类：  

![](https://i.imgur.com/2G9IQUF.png)  

在这里说明一下，由于本教程的行为树使用的是unity的状态机思想，所以BTDecorator和BTService类在本教程暂时用不到。

第四个C++类，是与任务有关的，继承BTTaskNode类，作为我们AI任务的基类：  

![](https://i.imgur.com/7DM9CSi.png)  

最后再建一个继承我们刚刚创建的EnemyTaskBase类的巡逻任务类：  

![](https://i.imgur.com/sDUiq23.png)  

4、由于我们新建的行为树里面不能直接指定C++类的黑板（BlackBoard），所以我们先在Content/Blueprint/Enemy下新建一个蓝图的黑板，继承SlAiEnemyBlackBoard：  

右击新建一个DataAsset：  

![](https://i.imgur.com/lwUDlqH.png)  

选择继承我们的C++类：  

![](https://i.imgur.com/kOxjM5N.png)  

命名为EnemyBlackboard：  

![](https://i.imgur.com/byowvEv.png)  

然后打开EnemyBehaviorTree，把右边的Blackboard Asset设置为这个：  

![](https://i.imgur.com/gPpx6WS.png)  

然后就可以关闭虚幻引擎，打开C++工程了。  


5、打开VS工程，来到SlAiTypes.h中，先定义行为树的枚举：  

![](https://i.imgur.com/Op9j48n.png)  

  
6、在EnemyBlackboard.h中，重写PostLoad方法：  

![](https://i.imgur.com/t0DRfJ8.png)  

实现之前，由于后面用到的类不容易通过自动提示得到，所以先在.cpp文件中添加所需要的头文件：  

![](https://i.imgur.com/nJH91qZ.png)  

然后实现PostLoad函数：   

![](https://i.imgur.com/x4U7GN7.png)  

![](https://i.imgur.com/Pk2CKu7.png)  

在行为树里面的变量，需要通过这种方式定义，具体类型就与UBlackboardKeyType后面的名字一致。第二个枚举，需要通过FindObject函数找到我们之前定义的EEnemyAIState枚举的反射。  
最后用内置的Keys把变量添加到黑板中。  

7、下面在EnemyTaskBase.h中定义初始化函数，并定义敌人相关指针：  

![](https://i.imgur.com/zhhTg1e.png)  

然后在.cpp文件实现：  

![](https://i.imgur.com/tX3h0w0.png)  

8、下面在写巡逻的C++逻辑之前，先运行虚幻引擎，把行为树的一些准备工作做好。  

打开EnemyBehaviorTree，在Root节点下添加一个Selector，然后在Selector下面添加7个Sequence，对应状态机的七个状态。  

![](https://i.imgur.com/n47SPvV.png)  


右击第一个Sequence，点击AddDecorator->Blackboard，给第一个Sequence添加一个判断器。 
选中此Condition，将右边的Blackboard Key改为EnemyState。并把Observer aborts属性设为Self。  

![](https://i.imgur.com/nO6pQcZ.png)  

![](https://i.imgur.com/r9qi0Ea.png)  

![](https://i.imgur.com/jOjyrS7.png)  

具体这些属性的解释，建议去官网查看文档：[http://api.unrealengine.com/CHN/Engine/AI/BehaviorTrees/NodeReference/Decorators/index.html](http://api.unrealengine.com/CHN/Engine/AI/BehaviorTrees/NodeReference/Decorators/index.html)   

9、接下来把剩下的6个节点都做与节点1一样的操作，只是把Key Value分别设置为Patrol下面的6个状态枚举。  
（其中把Attack和Escape两个节点的位置对调一下）：  

![](https://i.imgur.com/bFrOL4c.png)  

行为树的东西暂时先写到这里，下面的动作之后会添加。  

10、回到主界面，在Modes选项卡中找到Nav Mesh Bounds Volume，把它拖到场景中。并设置好位置和大小：  

![](https://i.imgur.com/s5sqMWz.png)  
这个组件是给AI做自动寻路用的，可以框定AI的范围。  

然后选中场景中的所有树，按Delete删掉：  

![](https://i.imgur.com/XvCjhPf.png)  

然后按P键，可以看到场景里面绿色的区域，都是AI可以走的地方。  

![](https://i.imgur.com/iQXu06h.png)  

Save一下，就可以关闭引擎了，准备写巡逻的逻辑。  



##24.2 运行行为树与实现巡逻AI  

1、我们先把行为树绑定到敌人的Controller上，然后再去实现巡逻类。  

首先打开EnemyController.h，在public下重写函数：   

![](https://i.imgur.com/BqeI9qg.png)  

还要在private下定义几个组件：  

![](https://i.imgur.com/TGAl43D.png)  

在实现之前，先在.cpp中添加所有需要的头文件：  

![](https://i.imgur.com/dGxHa4n.png)  

然后实现Possess函数：  

![](https://i.imgur.com/LnA7HOC.png)  

![](https://i.imgur.com/DHat4Ue.png)  

![](https://i.imgur.com/zD3V5QJ.png)  

![](https://i.imgur.com/ELUIV2j.png)  

这段代码就是获取我们在蓝图中做的行为树资源，并设置行为树的黑板。其中用到了两个Duplicate函数，将行为树和黑板都做了一下复制，这是因为如果只有一个行为树，在本工程中无法绑定很多的AI，复制一下是保险的做法。  

这段代码是参考RunBehaviorTree函数写出来的，想要深入了解一下的话，可以输入这个函数，并按住Ctrl点击左键去看它的源码。  


2、在行为树Start运行起来后，我们设置默认状态是巡逻。在StartTree函数后面直接添加代码：  

![](https://i.imgur.com/85cbXzR.png)  

3、接下来要修改敌人的移动速度。首先到EnemyCharacter.h中定义public函数：  

![](https://i.imgur.com/dSvFMZb.png)  

并在.cpp中实现：  

![](https://i.imgur.com/TlP7IM7.png)  

然后到EnemyController.cpp中，在刚刚设置预状态巡逻的代码后面再添加一行：  

![](https://i.imgur.com/AjFYCHL.png)  

这样就设置了初始速度。  

4、再把UnPossess实现：  

![](https://i.imgur.com/OzbUdXk.png)  

5、接下来我们要设置等待时的三个不同动作。  

先在EnemyAnim.h中定义三个protected变量：  

![](https://i.imgur.com/CN3S04A.png)  

并在类上面声明一下class：  

![](https://i.imgur.com/Tiu7MLQ.png)  

然后在public下定义函数：  

![](https://i.imgur.com/m8bJk0L.png)  

在实现之前，先到.cpp的构造函数中获取等待动作的资源：  

![](https://i.imgur.com/yEBDXmN.png)  

实现SetIdleType函数：  

![](https://i.imgur.com/K4oogDe.png)  

6、下面到EnemyCharacter.h中定义public函数：  

![](https://i.imgur.com/1IkAVWJ.png)  

然后在private下定义一个动作的引用：  

![](https://i.imgur.com/YF3O52r.png)  

并在.cpp的BeginPlay下实例化：  

![](https://i.imgur.com/E4NecXh.png)  

现在可以实现GetIdleType函数了：  

![](https://i.imgur.com/3TLkZex.png)  

在这里不仅让动作的类型随机，还让动作播放的次数也在1-3里面随机。  

7、准备工作都做好了，开始写巡逻的逻辑。  

在EnemyTaskWander.h中重写执行函数：  

![](https://i.imgur.com/rDdDyLp.png)  

然后定义两个protected变量：  

![](https://i.imgur.com/mhitGZX.png)  

发现WaitTime还没有在Blackboard黑板中定义，所以先去到EnemyBlackboard.cpp中再添加一个属性：  

![](https://i.imgur.com/PfJo930.png)  

回到EnemyTaskWander.cpp中，实现执行函数。  

方便起见，先把所需的头文件都包含一下：  
![](https://i.imgur.com/CIYlY53.png)  

然后实现执行函数：  

![](https://i.imgur.com/W1zrYAy.png)  

![](https://i.imgur.com/UiIo7ZC.png)  

![](https://i.imgur.com/SIMt6jg.png)  

8、生成无错误后，打开虚幻引擎，打开EnemyBehaviorTree行为树，在第一个条件节点下面添加三个Tasks：  

![](https://i.imgur.com/olN10gN.png)  

把场景里面之前调试用的那个Enemy删掉，重新再从C++类中拉进去一个。运行游戏， 可以看到敌人已经会巡逻了，有时是移动，有时是在某处站几秒，挥挥武器。  


（注：若打开虚幻引擎后发生崩溃，可以尝试删掉行为树和Blackboard两个文件，重新按照步骤创建好）  





