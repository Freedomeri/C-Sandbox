#第八节 UI动画实现与UI音乐播放与调节  

##8.1 UI动画实现  

1、首先把按钮菜单按钮的跳转函数实现：  

![](https://i.imgur.com/QIS3OaS.png)  
（输入switch的类型后，VA插件会自动把下面的case补全出来。）  

直接把我们之前写的ChooseWidget函数加到对应的case后面：  

![](https://i.imgur.com/NoEEQ9h.png)  

![](https://i.imgur.com/mMlkBNs.png)  

现在运行游戏看一下按钮的功能是否都实现了。  

2、定义与动画相关的变量：  

在SSlAiMenuWidget.h中定义private：  

![](https://i.imgur.com/wKfVCda.png)  

其中EMenu::Type是我们又在SlAiTypes.h中定义的动画枚举：  

![](https://i.imgur.com/fbaWuOf.png)  

3、实现初始化动画组件的函数  
先在MenuWidget.h中声明：  
![](https://i.imgur.com/KZh88LV.png)  

然后到.cpp文件中实现：  

![](https://i.imgur.com/XQ9Kssz.png)  

0.3秒开始播放动画，持续0.6秒。初始化一个播放器，然后通过AddCurve添加动画曲线来实现动画。  

由于会在这里我们将主界面初始化，并在播放动画的时候动态修改高度，所以可以删掉之前的两段代码：  
分别是ChooseWidget函数的最后一段  
![](https://i.imgur.com/GMxuxNe.png)  

和InitializeMenuList的最后一段  
![](https://i.imgur.com/ahuTCW8.png)  

写好初始化动画组件之后，就可以将这个函数添加到控件的构造函数最下边了：  
![](https://i.imgur.com/o0yXUXZ.png)  
 

4、在切换菜单方法里面，设置是否显示菜单的布尔值：
![](https://i.imgur.com/0jwpNMx.png)  

然后在触发按钮事件的方法中，判断按钮是否可点击：  

![](https://i.imgur.com/QB6DuzH.png)  
这是为了防止，当按下一个按钮的时候，迅速又按下另一个按钮而产生的冲突问题。我们要让按钮动画播放的期间，别的按钮不能被点击。  

5、在SSlAiMenuWidget.h下声明我们需要用到的函数：  

首先在public下重写每一帧的Tick函数：  

![](https://i.imgur.com/QhFFisN.png)  

然后在private下声明播放关闭动画的函数：  

![](https://i.imgur.com/wtXEQen.png)  

6、在.cpp中实现：  

![](https://i.imgur.com/67LJPwK.png)  
由于我们在初始化动画时，将MenuAnimation跳到了1，所以我们在播放按钮的关闭动画时，要反向播放，也就是从1到0。  

Tick函数：  

![](https://i.imgur.com/uyArNtX.png)  
  

在写其他情况之前，我们先将MenuItemOnClicked函数下的ChooseWidget函数全部改为PlayClose函数，然后理一下动画播放的思路：  

&emsp;&emsp;首先初始化控件组件和动画组件，让它显示主界面；然后假如我们点击开始游戏按钮，MenuItemOnClicked函数便会传入EMenuType::StartGame，然后设置播放状态为Close，马上播放一个反向动画。而在Tick函数中，由于AnimState变为了Close，就执行下面的内容，在实时修改Menu大小的函数中，获取动画当前的曲线值，乘以600（菜单的宽度），菜单的宽度便会实时变窄了。  

理解之后，继续在Tick中完善：  

![](https://i.imgur.com/9mMpBHl.png)  

然后完善case为Open的逻辑：  

![](https://i.imgur.com/0CFRqdg.png)  

为防止调试时出现bug，我们可以在没有设置动画的按钮上解锁按钮：  
![](https://i.imgur.com/V9JIC0C.png)  
EnterGame和EnterRecord同理。  

9、运行游戏，我们的按钮已经有切换的动画了。   


##8.2UI音乐播放与调节  

1、首先到SlAiMenuWidgetStyle.h中，继续定义几个声音相关的UPROPERTY：  

![](https://i.imgur.com/v8iA9CD.png)  

然后打开编辑器，打开BPSlAiMenuStyle，将刚刚的四个声音设置好：  

![](https://i.imgur.com/830zTu9.png)  

2、回到VS工程，在SlAiMenuWidget.cpp的构造函数下，实现背景音乐的播放：  

![](https://i.imgur.com/yKxzC9W.png)  

3、继续在此文件的下面找到PlayClose函数的实现，将下面代码添加到函数最后，实现点击按钮切换菜单时的音乐：  

![](https://i.imgur.com/75spmcy.png)  

4、到.h文件中，声明退出游戏的private函数：  

![](https://i.imgur.com/apkN512.png)  

然后回到.cpp文件中实现：  

![](https://i.imgur.com/ggGOCLk.png)  

Cast是非常常用的一个函数，用来从一个类型强制转换为另一个类型。参数中的UGamePlayStatics常用来获取游戏的Controller、GameMode等基础框架实例。  

5、要想正确播放声音，还需要一个TimerHanle（定时器）。而在Widget下面不能写TimerHandle，我们需要在SlAiHelper.h中SlAiHelper的namespace下写：  

![](https://i.imgur.com/B5uEhTy.png)  
首先播放传入的声音（后面会传入结束游戏的声音），然后定义一个定时器和定时器的委托。将委托绑定到定时器上，并把传入的方法绑定。最后从当前世界中获取并启动此定时器。  

![](https://i.imgur.com/3ORNhId.png)  

6、回到MenuWidget.cpp中：  

在按钮事件下延时调用退出函数：  

![](https://i.imgur.com/o1mIaTy.png)  

7、退出游戏的声音写好后，进入游戏的编写类似：  
先在SSlAiMenuWidget.h文件中声明函数：  

![](https://i.imgur.com/8ZulIEw.png)  

在.cpp文件中实现（由于进入游戏场景还没有写，先在这里输出一个Debug：  

![](https://i.imgur.com/RIwhZ0u.png)  

然后在按钮事件中调用：  

![](https://i.imgur.com/nPstCGc.png)  

现在可以先打开编辑器运行观察听一下效果（声音先开小一点）。  

8、调节音量  
音量的相关逻辑，我们写在DataHandle中。先在SlAiDataHandle.h中定义private变量：  

![](https://i.imgur.com/BAx4jUj.png)    

然后声明一个private函数用来初始化声音：  

![](https://i.imgur.com/AjuMg7P.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/U5NjhzQ.png)  

写完这个函数就可以把它添加到构造函数中了：  

![](https://i.imgur.com/m92KTeL.png)

还需要在ResetMenuVolume函数中循环设置音乐音量：  

![](https://i.imgur.com/7fAz9Gt.png)  

音效同理：  
![](https://i.imgur.com/lr8SoGr.png)  


打开编辑器，运行游戏，已经可以正常调节音量了。  









  

