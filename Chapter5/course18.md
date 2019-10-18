#第十八节 玩家状态UI、游戏暂停与输入模式切换  

##18.1 玩家状态UI  

这一小节我们制作的玩家状态UI在蓝图里面是这样的效果：  

![](https://i.imgur.com/5P36eB5.png)  

对应的蓝图结构如下：  

![](https://i.imgur.com/GOV97kL.png)  

1、打开虚幻编辑器，创建新的Slate控件：  

![](https://i.imgur.com/T4B0WOG.png)  

2、打开VS工程，去到SSlAiGameHUDWidget.h中，定义public指针：  

![](https://i.imgur.com/G2KR3wg.png)  

然后在.cpp文件中将此控件添加到其中：  

![](https://i.imgur.com/HiUK5O1.png)  
位置在与准星同级的部分。  

3、然后开始编写状态UI：  
首先是与所有Widget一样的获取GameStyle：  

![](https://i.imgur.com/JSeMhIP.png)  

![](https://i.imgur.com/fz46joC.png)  

然后开始在ChildSlot中添加组件：  

![](https://i.imgur.com/oaqE3MM.png)  

写到这儿我们需要去定义一个背景的笔刷：  
去到GameWidgetStyle.h中：  

![](https://i.imgur.com/KxJCYU5.png)  

然后回到PlayerStateWidget.cpp中，继续编写组件：  

![](https://i.imgur.com/1MgLg7J.png)  
Offset中的前两个参数是位置，后两个是大小。

4、写到这需要添加血条了。先在.h文件中定义private血条指针：  

![](https://i.imgur.com/nc8SUdD.png)  

然后回到.cpp文件，创建血条：  

![](https://i.imgur.com/1Zsc5j8.png)  

背景图片设为空是因为，之后我们的整体状态背景框已经做好了，只需要设置里面的填充图片即可。  

5、接着是饥饿度的条：  
写在与血条平级的位置：  

![](https://i.imgur.com/PNxVs0q.png)  

6、接下来是人物头像和背景：  

![](https://i.imgur.com/6NrCESJ.png)   


![](https://i.imgur.com/7F0mWh3.png)  

这样显示部分已经写完了。  

7、我们需要定义函数，用来动态修改血量以及饥饿度的值：  
先在PlayerStateWidget.h中定义public函数：  

![](https://i.imgur.com/7PVfGX5.png)  

然后到.cpp文件中实现：  

![](https://i.imgur.com/IQ79kMK.png)   

8、去到PlayerState.h中定义动态修改的委托：  
写在UCLASS之上：  

![](https://i.imgur.com/E1Ldgfj.png)  

定义委托变量(public)：  

![](https://i.imgur.com/HABlF1V.png)  


9、在GameHUD.cpp中绑定委托：  

![](https://i.imgur.com/3SNK1N7.png)  

10、下一步，在PlayerState.h里面定义血量和饥饿度的private变量：  

![](https://i.imgur.com/NRjGvFC.png)   

并在.cpp的构造函数中赋初值：  

![](https://i.imgur.com/SZkV4dS.png)  

11、要在Tick函数中实现慢慢减少饥饿度，首先将Tick函数定义在public:  

![](https://i.imgur.com/vJTfMid.png)  

在实现之前，记得在构造函数中设置允许每帧调用：  

![](https://i.imgur.com/gwtKBzY.png)  

然后实现Tick函数：  

![](https://i.imgur.com/BU7PSWB.png)  
  

![](https://i.imgur.com/NYiYzyR.png)  
其中Hunger也除以500是因为，要让饥饿度满的时候，保持一段时间，再开始减饥饿度。  

我们的C++代码就基本上写完了。  


12、打开虚幻编辑器，运行之前先在Material文件夹下创建一个新材质：  

![](https://i.imgur.com/6OGkjHk.png)  
打开此材质  
然后在资源文件夹下找到这个贴图文件，选中  

![](https://i.imgur.com/C47P5XV.png)  

在新材质中按住T键点击鼠标左键，创建一个TextureSample，再用同样的方法创建第二个。并把两个的Texture设为刚才选中的贴图：  

![](https://i.imgur.com/bMbiINR.png)  

![](https://i.imgur.com/xwLQrgk.png)  

右击上面的TextureSample，将它提升为变量，并设置名称为TargetTex：  

![](https://i.imgur.com/GmBFkMs.png)  

![](https://i.imgur.com/ICkxKlk.png)  

再把右边的PlayerStateMat节点的Material Domain设为User Interface，Blend Mode设置为Masked：  

![](https://i.imgur.com/WYzq9VC.png)  

将蓝图连成下图所示：  

![](https://i.imgur.com/7jQMw0u.png)  

13、回到Material文件夹，右击新材质，创建材质实例Instance：  

![](https://i.imgur.com/vm9Op2g.png)  

打开材质实例，把TargetTex属性设置为PlayerHead： 


![](https://i.imgur.com/YIzN4Ki.png)  

再新建一个材质实例PlayerHeadBGMatInst：  

![](https://i.imgur.com/wTbs12M.png)  

打开此实例，将TargetTex设置为PlayerHeadBG： 
 
![](https://i.imgur.com/ugfUC6y.png)  

14、去到Style文件夹下打开BPSlAiGameStyle，把通过C++新创建的几个Brush设置好：  

![](https://i.imgur.com/m0r9pl8.png)  

![](https://i.imgur.com/qkIfRTf.png)  

![](https://i.imgur.com/9e1ebjy.png)  

![](https://i.imgur.com/yNqGB2v.png)  

![](https://i.imgur.com/TDYG2tk.png)  

15、运行游戏，可以看到人物状态UI了：  

![](https://i.imgur.com/KwRhbtW.png)  



##18.2 游戏暂停与输入模式切换  

这一小节我们模仿我的世界，按ESC弹出暂停界面。然后实现背包和聊天框UI显示。   

1、首先打开虚幻编辑器，在项目设置里面的Input菜单下，添加三个个新的Action，分别为暂停，背包和聊天室：  

![](https://i.imgur.com/80gEuHP.png)  

2、新建三个C++文件，全部继承Slate Widget。  

![](https://i.imgur.com/DIArsOx.png)  

![](https://i.imgur.com/xEq8dPr.png)  

![](https://i.imgur.com/4hp6UHN.png)  
第三个背包UI放在Widget下的新文件夹Package下，因为还有一些与背包有关的类需要之后创建。   

3、打开VS工程，打开SSlAiGameHUDWidget.h，先将我们创建的三个SlateWidget在public下定义：  

![](https://i.imgur.com/zexHfzo.png)  

还需要在private下定义一个黑色遮罩的UI指针  

![](https://i.imgur.com/BBuv7ba.png)  

然后到.cpp文件中在总的SOverlay下添加黑色遮罩控件：  

![](https://i.imgur.com/TvEQXX5.png)  

运行过游戏的话可以清楚的知道，这个黑色遮罩就是在暂停时让背景暗一点的效果。   

4、接下来把三个UI控件添加到主界面中：  

![](https://i.imgur.com/jBQxHwe.png)  

![](https://i.imgur.com/t3PayS7.png)  

![](https://i.imgur.com/7pzL9Xm.png)   
Visibility属性设为Hidden就是让它一开始不显示。  


5、在SlAiTypes.h中，定义游戏界面UI的枚举：  

![](https://i.imgur.com/JrOjKEu.png)  

6、回到GameHUDWidget.h，定义一个private的UIMap，用于盛放三个UI控件：  

![](https://i.imgur.com/cSrs5iC.png)  

并定义private初始化函数：  

![](https://i.imgur.com/6qYesLS.png)  

在.cpp文件实现：  

![](https://i.imgur.com/NxRCPAf.png)  

并放在构造函数最后调用：  

![](https://i.imgur.com/XvANLfV.png)  

7、在GameHUDWidget.h文件中定义pubic函数，用于之后的PlayerController的委托绑定

![](https://i.imgur.com/qNVTvRH.png)  

8、在PlayerController.h中定义委托：  

![](https://i.imgur.com/Q5kGtCY.png)  

定义相应的委托变量：  

![](https://i.imgur.com/ckd7IV7.png)  

去到GameHUD.cpp中绑定委托：  

![](https://i.imgur.com/qCbbzdB.png)  

9、来到PlayerController.h下，定义private变量：  

![](https://i.imgur.com/ZMtsM6v.png)  

10、定义三个privateUI事件函数：  

![](https://i.imgur.com/Y1M1D1V.png)  

11、实现三个事件之前，先在SetupInputComponent函数中绑定按键事件：  

![](https://i.imgur.com/v5YcBYS.png)  
bExecuteWhenPaused设为true，是为了让游戏暂停的时候依然可以按Esc键返回游戏。否则游戏暂停时不会相应这些Action。  

12、在BeginPlay下把默认UI状态设为Game：  

![](https://i.imgur.com/Ge1oPm5.png)  

13、接下来实现EscEvent：  

![](https://i.imgur.com/DgjtbAU.png)  

先搭好框架。下面三个case放在一起是因为，暂停、背包、聊天室的界面下，按Esc，触发的事件是一样的。  

然后回到.h文件，定义一个切换输入模式的方法：  

![](https://i.imgur.com/hgRBu0G.png)  

先将此函数实现：  

![](https://i.imgur.com/x6z8Xnt.png)   

![](https://i.imgur.com/1gMFnIf.png)  

最后再实现EscEvent：  

![](https://i.imgur.com/248pLJv.png)  

![](https://i.imgur.com/uPC1rc3.png)  

14、运行游戏，按下Esc键已经可以随时暂停游戏了。  


