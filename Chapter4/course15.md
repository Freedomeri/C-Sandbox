#第15节 动作通知与资源  

##15.1 动作通知与读取资源Json  

1、在上节的调试overlap检测成功之后，可以先在HandObject.cpp中把碰撞关掉了：  

![](https://i.imgur.com/iULiRfh.png)  
然后到PlayerCharacter.cpp中，把ChangeHandObject函数的下面部分删掉：  

![](https://i.imgur.com/HO08Rmc.png) 

并将此函数参数赋的nullptr值去掉。  


2、我们想要控制物品与场景的交互，首先在SlAiPlayerAnim.h中定义public函数：  

![](https://i.imgur.com/36AsPh7.png)  

我们要获取Character当前手上的物品类，还要在SlAiPlayerCharacter.h中定义函数：  

![](https://i.imgur.com/KBBKLIt.png)  

想要暴露出手持物品类的碰撞属性，需要再去到HandObject.h中定义函数：  

![](https://i.imgur.com/SHMNHNz.png)  

并在.cpp文件中实现：  

![](https://i.imgur.com/8rC6BIE.png)  

接着回到PlayerCharacter中，实现ChangeHandObjectDetect函数：  

![](https://i.imgur.com/9lXjew2.png)  

最后回到PlayerAnim.cpp中实现ChangeDetection函数：  

![](https://i.imgur.com/leoMFiV.png)  
稍后我们会在动画蓝图中调用PlayerAnim的此函数。  

整体思路是，首先动画蓝图调用方法，传入参数决定是否开启碰撞，然后告诉Character调用函数获取手持物品的bGenerateOverlapEvents并设为参数的值。  


3、打开虚幻编辑器，打开第三人称动画蓝图，我们在EventGraph中添加蓝图节点：  

![](https://i.imgur.com/OT6xPf4.png)  

这两个事件是我们在动画中提前在动画的合适位置添加好的通知。在右下角AssetBrowser中随便打开一个动画文件，可以看到通知：  

![](https://i.imgur.com/fm8ciNU.png)  

4、用同样的方法，也把第一人称动画蓝图设置好：  

![](https://i.imgur.com/6aW1Tdd.png)  

保存好后，运行游戏，现在我们在碰到墙的时候不会直接触发事件了，而我们点击左键砍的时候才会触发。  

![](https://i.imgur.com/O0fNjYZ.jpg)  

5、我们实际想要产生交互的肯定不是这个墙，而是资源。下面我们就在c++中读取资源。  

首先我们去到ConfigData文件夹看一下资源的Json文件：  

![](https://i.imgur.com/7fbVXSB.png)  

这里面的属性有中英文名，资源类型，血量，还有掉落的物品。其中掉落物品的数据格式第一位是物品ID，第二位和第三位分别是最小掉落数和最大掉落数。比如这个图片中的掉落物是木头和苹果，最少掉3个木头2个苹果。  

6、回到VS工程，首先在SlAiTypes.h中定义资源枚举：  

![](https://i.imgur.com/FlOOmAQ.png)  

然后继续定义一个资源属性的结构体：  

![](https://i.imgur.com/vYBb3Iw.png)  

还有构造函数：  

![](https://i.imgur.com/UMBi8B5.png)  

关于这个整形数组的数组，里面的数组保存的是物品序号、掉落数的信息，外面数组保存的是有几个掉落物的信息。  

类似于物品类型，我们需要在SlAiJsonHandle.h中定义private转换函数：  

![](https://i.imgur.com/wu1gwqu.png)  

并在.cpp文件中实现：  

![](https://i.imgur.com/e1G5rCQ.png)  

7、定义解析资源函数：  
在JsonHandle.h中定义public资源解析函数：  

![](https://i.imgur.com/F1n0idp.png)  

还要定义一个private名称变量：  

![](https://i.imgur.com/D0Pp6Q4.png)  

并在构造函数中赋初值：  
![](https://i.imgur.com/yidrpF9.png)  

然后仿照之前的解析函数，实现资源解析函数：  

![](https://i.imgur.com/QSOMvJA.png)  

![](https://i.imgur.com/VrG4AK0.png)  

![](https://i.imgur.com/sy7cnvP.png)  

![](https://i.imgur.com/kCuQTOu.png)  

其中，含有物品ID和数量的那个数组，我们通过_和,符号拆分出来，分别保存。并用FCString::Atoi方法把字符串转化为int值。  

8、下一步需要到DataHandle中调用此函数，先在.h文件中定义private初始化方法：  

![](https://i.imgur.com/5nhRz5Z.png)  

再定义一个public的资源的Map：  

![](https://i.imgur.com/G6h2tad.png)  

去到.cpp文件中实现初始化资源函数：  

![](https://i.imgur.com/wmnbbkW.png)  

并把此函数放到初始化游戏数据函数中：  

![](https://i.imgur.com/MXVx7lh.png)  

9、最后为方便调试观察，我们仿照物品数据的Debug，在SlAiTypes的资源数据结构中添加ToString方法：  

![](https://i.imgur.com/2KN24Hd.png)  

并在DataHandle.cpp中将它Debug出来：  

![](https://i.imgur.com/aTBqxfy.png)  

打开编辑器，运行游戏，可以看到效果了：  

![](https://i.imgur.com/jL6kn4b.png)  

10、开始创建资源的C++文件：  
在Public下创建一个Resource的文件夹，并在下面新建第一个资源基类，继承Actor：  

![](https://i.imgur.com/8u8mzgr.png)  

然后基于这个基类，分别创建岩石类、树木类  

![](https://i.imgur.com/0h83uTw.png)  

![](https://i.imgur.com/GCmohQ9.png)  




##15.2 资源与可拾取物品  

1、首先在编辑器中类似于Player和Tool一样，创建碰撞的配置文件：  
新建一个ObjectChannel：  

![](https://i.imgur.com/dTWsQIc.png)  

再新建一个Preset配置文件：  

![](https://i.imgur.com/xR8RTO2.png)  

同时要记得打开ToolProfile，将里面的Resource改为Overlap。  

2、回到VS工程，正式编辑资源类的代码。  

先到ResourceObject.h中，添加需要的变量：  

![](https://i.imgur.com/We2iT5s.png)  

![](https://i.imgur.com/bLAVjhz.png)  

然后到.cpp文件，在构造函数中实例化变量：  

![](https://i.imgur.com/Zmc0WDk.png)  

继续添加碰撞属性：  

![](https://i.imgur.com/FqScoOg.png)  

3、接下来是每一个子类实现随机资源：  
树：  

第一步.h文件中定义构造函数：  

![](https://i.imgur.com/TQfh3pQ.png)  

第二步在.cpp中添加所有树资源的路径（可复制源码）：  

![](https://i.imgur.com/KPlgsA5.png)  

![](https://i.imgur.com/PrfsLej.png)  
  

然后再写模型的随机：  
 
![](https://i.imgur.com/asrWiVU.png)  
不能用静态变量是因为，每次实例化的时候，模型都应该不同，而不是整个类都用一个模型。

这样，树的类就写完了。  

岩石：  

与树的步骤相同：  

![](https://i.imgur.com/XMUZUJQ.png)  

![](https://i.imgur.com/1NQ9SGu.png)  

![](https://i.imgur.com/Ti1KWdu.png)  


运行游戏，拖动我们的C++类到场景中，可以看到会随机生成不同模型了。  

4、下面我们做可拾取物品（木头和石块）。  
首先再添加一种Collision：  

![](https://i.imgur.com/h3N0aUX.png)  

![](https://i.imgur.com/m0mRGyJ.png)  


然后还要把之前新建的几个再设置一下，这里只需要再把ToolProfile的配置文件的PickUp改为Overlap就可以了。  

  
5、接下来开始建可拾取物品的C++类：  
在Public文件夹下新建Pickup文件夹，并新建继承Actor的基类：  

![](https://i.imgur.com/U3xad8U.png)  

然后继承此基类，分别创建木头和石头的类：  

![](https://i.imgur.com/Oar4iUr.png)  

![](https://i.imgur.com/BoKA0Nl.png)  

6、C++类创建好之后，去到VS工程，先实现基类。  

和资源类相同，先在PickupObject.h中定义组件指针：  

![](https://i.imgur.com/h3C4C2B.png)  

再定义一个物品序号：  

![](https://i.imgur.com/TGoUkzK.png)  


然后到.cpp文件中，在构造函数中实例化组件：  

![](https://i.imgur.com/yNLG3Ob.png)  

7、基类实现好之后，去实现子类：  
可拾取石头：  
在PickupStone.h中创建public构造函数，并在.cpp中实现：  

  
![](https://i.imgur.com/5J10In9.png)  


可拾取木头：  
与石头同理：  

![](https://i.imgur.com/aIxYXle.png)  


8、打开虚幻编辑器，接下来要制作的是UI界面的一个控件，当玩家射线检测到某个物品时，在UI上显示文字，如图：  

![](https://i.imgur.com/yGOk6EU.png)  


首先我们新建一个C++类，继承SlateWidget：  

![](https://i.imgur.com/ESctiy2.png)  

然后关闭虚幻4，回到VS，首先需要在GameHUDWidget.h下定义一个public共享指针，用来保存我们的信息框UI控件：  

![](https://i.imgur.com/i3KfJHg.png)  

然后在.cpp文件中将此控件添加进去。在SOverlay的范围内再加一个Slot插槽，让此控件与上面的快捷栏同级：  

![](https://i.imgur.com/gmpxxuW.png)  


9、开始编写RayInfoWidget。先在.h文件中定义private变量：  

![](https://i.imgur.com/0JKIQ0P.png)  
记得要在上面声明一下class STextBlock;

然后到.cpp文件中：  

![](https://i.imgur.com/kQ5HaSV.png)  

写到这里发现没有定义背景图片的笔刷，我们去到SlAiGameWidgetStyle.h中添加笔刷：  

![](https://i.imgur.com/aifbJZj.png)  

回到RayInfoWidget中，完成剩下的控件：  

![](https://i.imgur.com/YrBb4Yh.png)  

10、我们需要让Controller实时检测更改文本框里的文本，所以在RayInfoWidget.h中需要定义一个委托：  

![](https://i.imgur.com/yUnIrp9.png)  

然后定义public委托变量：  

![](https://i.imgur.com/DchVCoH.png)  

11、重载一下射线信息控件的帧函数(public)，并定义private布尔变量：  

![](https://i.imgur.com/nf6hERi.png)  

![](https://i.imgur.com/6iRyFVv.png)  

在.cpp中实现：  

![](https://i.imgur.com/y0HjCbU.png)  
在Tick的第一帧调用委托，注册到PlayerState中。   

12、在PlayerState.h中定义public事件函数，供委托调用：  

![](https://i.imgur.com/3UsqhO7.png)  



在实现函数之前，我们可以参照快捷栏的实现，首先要定义一个FText类型的TAttribute(private)：  

![](https://i.imgur.com/xU15Gzn.png)  

然后定义获取函数(private)：   

![](https://i.imgur.com/vl2hAxt.png)  

先给这个获取函数返回一个hahaha的文本，以便观察调试：  

![](https://i.imgur.com/6PqrWRC.png)  


最后去实现那个委托事件函数：  

![](https://i.imgur.com/h4AsP6w.png)  

13、最后去到GameHUD.cpp中，将委托与函数绑定。  

在BeginPlay函数中添加：  

![](https://i.imgur.com/0WPEMqQ.png)  


14、打开编辑器，我们在BPSlAiGameStyle样式文件中将刚才定义的背景笔刷设置为Menu菜单按钮的背景：  

![](https://i.imgur.com/WET6fqb.png)  

运行游戏，效果如下：  

![](https://i.imgur.com/8eupGkn.png)  



