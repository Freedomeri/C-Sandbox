#第九节 场景跳转及添加玩家模型与相机  

##9.1 场景跳转与GameInstance  

1、首先我们打开编辑器，找到Res资源文件夹下的游戏场景地图，如图所示：  

![](https://i.imgur.com/WFSkLff.png)  

将此地图移动到我们自己的Map文件夹下，选择Copy。  

![](https://i.imgur.com/dwgbNuq.png)  

复制过去之后，将名字改为GameMap。  

![](https://i.imgur.com/eA3Nkpc.png)  

2、关闭编辑器，打开VS工程，去到MenuWidget.cpp下，找到按钮按下事件中的EnterGame，添加下列代码：  

![](https://i.imgur.com/G6cKBdN.png)  
这里用到了我们之前在NewGameWidget中写的函数，判断输入的新存档名是否可以使用，若可以则进入游戏。  

同理，EnterRecord中也是这样，只不过选择的是现有存档，不需再判断：  
![](https://i.imgur.com/I6sQjl3.png)  


3、在EnterGame函数中实现场景跳转：  

![](https://i.imgur.com/f2ymR9K.png)  

（此为4.19引擎的写法，其他版本引擎可能需要下面的写法：）  

![](https://i.imgur.com/1tvvRWc.png)  


4、打开虚幻编辑器，我们想要看到切换效果的话，首先要在GameMap里面添加一个PlayerStart。  
![](https://i.imgur.com/ucXxG8F.png)  
将它拖动到场景中任意一处亮一些的地方，以便我们能看清周围。  

![](https://i.imgur.com/tvFFy37.png)  

然后双击回到MenuMap场景，点击Play，然后按照正常的流程，进入新游戏，我们就可以看到切换场景的效果了：  

![](https://i.imgur.com/0a37p5G.jpg)  

到此，我们的菜单部分算是基本完成了。  


5、搭建GameMap的框架：  

首先为方便调试，将项目设置里面的默认关卡改为GameMap：  

![](https://i.imgur.com/ytbyF8I.png)  

然后开始新建C++类，首先是继承GameModeBase的游戏模式类：   
![](https://i.imgur.com/2w1RG4W.png)  

第二个是HUD类：  

![](https://i.imgur.com/AvyIzmu.png)  

第三个是GameInstance（记得在选择父类时，要点击ShowAllClasses）：  

![](https://i.imgur.com/LRv0iY6.png)  


创建完成后，打开GameMap关卡，在右边的WorldSettings窗口下，设置GameMode：  

![](https://i.imgur.com/4BqRx4t.png)  

在Project Settings下，将GameInstace也设置为我们刚刚创建的Instance：  

![](https://i.imgur.com/u0wfrUn.png)  

6、回到VS工程，在SlAiGameInstance.h中定义下列属性：  

![](https://i.imgur.com/Ggh5dLd.png)  

在SlAiMenuGameMode.h中定义BeginPlay：  
![](https://i.imgur.com/wlhjvPh.png)  

并在.cpp中实现：  

![](https://i.imgur.com/w92lKo2.png)  

这样就将GameInstance里面的游戏名在一开始就设置好了。  

7、在SlAiGameMode.h中添加构造函数和BeginPlay：  

![](https://i.imgur.com/ZTmCiyI.png)  

并在.cpp里面实现BeginPlay：  
![](https://i.imgur.com/qy6uo6b.png)  

接下来打开虚幻编辑器，测试跳转场景后是否能保存着两个值：  

在我们输入存档名，点击进入游戏后，出现如下画面：  

![](https://i.imgur.com/8G0SGud.png)  

说明存档名和游戏名是可以在跳转场景后保存的。  

8、创建继承PlayerController的玩家类：  

![](https://i.imgur.com/96YdyiS.png)  

创建继承Character的角色类：  

![](https://i.imgur.com/qzVVou2.png)  

再创建继承PlayerState的玩家状态类：  

![](https://i.imgur.com/bA1IyHY.png)  

还要创建一个继承Anim Instance的动作类（需要ShowAllClasses）：  

![](https://i.imgur.com/5HAZ5XP.png)  

然后基于这个动作类，分别创建两个继承自它的第一人称、第三人称动作基类：  

![](https://i.imgur.com/J0mwvFr.png)  

![](https://i.imgur.com/ajQhAbm.png)  

![](https://i.imgur.com/Msxxu00.png)  

9、所需的类创建好后，我们到VS工程下开始指定GamePlay框架的内容：  

首先来到SlAiGameMode.cpp，添加组件：  

![](https://i.imgur.com/2datrZd.png)  
需要引入的头文件，自行根据提示快速添加。  

10、在SlAiGameMode.h中重写Tick函数：  

![](https://i.imgur.com/UnCvJFs.png)  

在.cpp文件实现之前，先在构造函数中添加语句：  

![](https://i.imgur.com/Xnt5wpM.png)  

在继承自Actor的类（A开头）中，最好在构造函数中加上这一句，确保Tick帧函数的执行。  



##9.2 添加玩家模型与相机  

1、在用代码添加模型和相机前，先了解一下ue4添加组件的一些基础知识：  

![](https://i.imgur.com/kFTigu1.png)  

其中第一个方法，相当于在蓝图中的Add Component按钮：  

![](https://i.imgur.com/iHFfwKr.png)  

第二个方法，相当于在蓝图的EventGraph面板里通过蓝图节点的方式添加组件。  

关于资源加载：  

![](https://i.imgur.com/eNur6og.png)  

2、回到VS工程，在SlAiPlayerCharacter.cpp的构造函数中添加下列代码：  

![](https://i.imgur.com/D3685Dr.png)  

其中的资源文件路径为：  

![](https://i.imgur.com/7Al6YBX.png)  
我们用鼠标点击选中Player资源文件之后，通过Ctrl + C，可复制路径，然后在上面的TEXT里面Ctrl + V，即可将路径直接粘贴到代码中。  

3、为方便看到C++代码的实时效果，我们在虚幻编辑器的Content目录下新建Blueprint/Temp文件夹，用来存放继承我们C++类的蓝图：  

![](https://i.imgur.com/Od9yo9X.png)  

在实现C++代码后，要想在编辑器中同步，首先要点击Compile按钮：  

![](https://i.imgur.com/aADLCr4.png)  
   
然后在里面新建蓝图，继承SlAiPlayerCharacter类：  

![](https://i.imgur.com/4FymGea.png)  

打开此蓝图，可以看到我们添加的骨骼模型已经成功了。  

4、回到VS工程，继续在刚才的代码下面添加Mesh的属性：  

![](https://i.imgur.com/fcF12KI.png)  

其中，bOnlyOwnerSee：玩家本人才能看到Mesh；  
bReceivesDecals：不接受其他物体的反射效果，如影子等；  
SetCollisionObjectType：设置碰撞属性为Pawn类型；  
SetCollisionEnabled：关闭碰撞（我们只让模型的根组件有碰撞）；此属性对应的蓝图部分为：  

![](https://i.imgur.com/buqFcaN.png)  
  
SetCollisionResponseToAllChannels：忽略此组件与其他所有物体的碰撞交互与事件；  
最后两个是设置相对位置和角度，让此模型向下95单位，绕Z轴旋转-90度。  

设置完成后可以打开虚幻编辑器，查看模型是否已经调整合适。  

5、有了第三人称的Mesh，我们再添加一个第一人称的Mesh：  
先在SlAiPlayerCharacter.h里面定义属性：  

![](https://i.imgur.com/QG3KKKN.png)  
VisibleDefaultsOnly参数表明，此属性在蓝图中只可见，不可更改。之前的EditAnywhere属性不能放在private下，只能在public或protected下。  

回到.cpp文件，在构造函数中继续添加：  

![](https://i.imgur.com/x524krO.png)  
这里的资源文件，根据图片路径自行找到并复制粘贴。  
SetupAttachment：将此组件添加到根组件下。  
bCastDynamicShadow：不渲染此模型的影子（第一人称只渲染两个手臂，不需要渲染影子）。  

![](https://i.imgur.com/S25uZpd.png)  


打开编辑器，打开PlayerCharacterBP蓝图，可以看到我们添加的MeshFirst，对应模型只有两个手臂（黄色高亮部分）：  

![](https://i.imgur.com/c5aq5TK.png)  

6、添加相机：  
在SlAiPlayerCharacter.h文件中定义三个相关UPROPERTY属性：  

![](https://i.imgur.com/y1dFQhI.png)  

然后在.cpp文件中实例化：  

![](https://i.imgur.com/CKmrL5g.png)  
让它跟随人物的旋转而旋转。  

![](https://i.imgur.com/XRwusG4.png)  
这里设置第三人称相机不跟随旋转是因为，它已经绑定到摄像机手臂上了，再次旋转会转两倍的角度。   

7、打开虚幻编辑器，重新创建一个继承SlAiPlayerCharacter的蓝图，可以看到代码的效果：  

![](https://i.imgur.com/WYkKvfU.png)  

![](https://i.imgur.com/ipU2zYe.png)  




