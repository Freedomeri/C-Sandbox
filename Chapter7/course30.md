#第三十节 聊天信息显示、聊天室与敌人血条修正  

##30.1 聊天信息显示：  

聊天相关的UI控件，一个是之前创建过的ChatRoom，是我们打字输入的那个框。另一个我们要新建一个ChatShow，用来显示所有的聊天信息。  

1、打开虚幻引擎，在Widget文件夹下新建C++文件：  

![](https://i.imgur.com/b43hCeZ.png)  

关闭虚幻引擎，来到VS工程。  

2、来到GameHUDWidget.h下，定义一个private变量：  

![](https://i.imgur.com/ij7uVxU.png)  

然后到.cpp文件，在黑色遮罩上面，小地图下面添加一个Slot，插入我们的聊天显示栏：  

![](https://i.imgur.com/zzQ5emU.png)  

3、打开ChatShowWidget，老样子先把GameStyle获取一下。  

![](https://i.imgur.com/FFNDWvs.png)  

![](https://i.imgur.com/7pbHntv.png)  

在正式编写UI布局前，先在.h文件中private下定义所需控件指针：  

![](https://i.imgur.com/NcMn2ST.png)  

然后到.cpp文件中写一个UI的大致框架：  

![](https://i.imgur.com/dYOmWNW.png)  

4、在聊天显示的信息方面，我们定义一个结构体来作为每条信息的结构：  
在头文件下面定义结构体ChatShowItem：  

![](https://i.imgur.com/KqO0sBJ.png)  

透明值是用来实现字体的慢慢消失效果的。  

然后是它的构造函数：

![](https://i.imgur.com/1XRhI9f.png)  

![](https://i.imgur.com/sMXTL0W.png)  

不明白布局的可以先打开源工程，看一下左下角聊天框的UI布局。  

5、接下来在结构体中再定义几个与聊天信息显示相关的方法：  

![](https://i.imgur.com/mFqNNyQ.png)  

![](https://i.imgur.com/rGDxYbz.png)  


6、聊天信息的结构体写好了，我们来到ChatShowWidget.h中，在public下重写一下Tick函数：  

![](https://i.imgur.com/XbJrjlS.png)  

再添加两个相关函数，一个public一个private：  

![](https://i.imgur.com/d9prsJH.png)  

还要定义两个private数组：  

![](https://i.imgur.com/33Id0bJ.png)  
不要忘记在最上面声明一下结构体：  

![](https://i.imgur.com/BCTFuzL.png)  


7、到.cpp文件中，先把InitializeItem函数添加到构造函数最后：  

![](https://i.imgur.com/SmzeUmT.png)  

然后实现此初始化函数：  

![](https://i.imgur.com/nyOSZ6w.png)  

再继续实现AddMessage函数：  

![](https://i.imgur.com/1B1C95L.png)  

![](https://i.imgur.com/HRxVUhK.png)  

![](https://i.imgur.com/aPoVimX.png)  
  

8、接下来是Tick函数，我们要在每一帧都更新聊天信息框的内容，并且更新激活和未激活的两个列表：  

![](https://i.imgur.com/RVIf3VE.png)  

![](https://i.imgur.com/d3FrXq0.png)  

9、由于此项目为单机，不涉及联网，所以我们只能模拟聊天功能。  
来到GameHUDWidget.h中，定义private变量：  

![](https://i.imgur.com/7ZfI4uN.png)  

我们想让此计时器加上DeltaTime，然后每过几秒就显示一条消息。为此需要重写Tick函数：  

![](https://i.imgur.com/c58B5bu.png)  

来到.cpp文件中，首先把消息计时器在InitUIMap函数中初始化：  

![](https://i.imgur.com/AiWadci.png)  

然后实现Tick函数：  

![](https://i.imgur.com/7V0tRN5.png)  

![](https://i.imgur.com/fVMk5M9.png)   
  

10、打开虚幻引擎，运行游戏，可以看到左下角已经在显示信息了：  

![](https://i.imgur.com/0GU2sFo.png)  


##30.2 聊天室与敌人血条修正  

1、我们这一小节实现玩家的聊天室功能。打开ChatRoomWidget.h，与其他Widget一样，先获取GameStyle：  

![](https://i.imgur.com/urYDoEH.png)  

![](https://i.imgur.com/TtSoF6t.png)  

然后把.cpp文件之前写的控件先删掉，重新编写控件：  

先写个大致框架：  
![](https://i.imgur.com/LY3cUQY.png)  

写到这里需要去到GameWidgetStyle.h中定义笔刷：  

![](https://i.imgur.com/3avDE5F.png)  

然后回到ChatRoomWidget，继续编写控件代码：  

![](https://i.imgur.com/Uc9v7Lf.png)  

在插槽中第一个子控件是一个滚动框，我们先到.h中定义private指针：  

![](https://i.imgur.com/Zya7zFZ.png)  

回到.cpp中：  

![](https://i.imgur.com/Kd2MBBC.png)  
  

写到这里需要在.h文件中定义一个private输入框的指针：  

![](https://i.imgur.com/BCuzBMG.png)  

回到.cpp文件：  

![](https://i.imgur.com/siZRiJk.png)  

总共有两个Overlay控件，第二个Overlay用来盛放下面的输入框和发送按钮。  


2、我们需要实现按Enter键提交信息的功能，来到.h文件，定义public函数：  

![](https://i.imgur.com/wy09SF0.png)  

然后回到.cpp中，注册提交事件，并添加一个发送按钮的插槽：  

![](https://i.imgur.com/M7fQZ35.png)  

我们需要给按钮绑定事件，先到.h文件声明public函数：  

![](https://i.imgur.com/odHCiWr.png)  

然后在Button下绑定：  

![](https://i.imgur.com/l1r42FC.png)  

3、显示部分UI布局暂时写到这里，接下来和ChatShow控件类似，需要定义一个信息的结构体：  

在.cpp文件头文件下面定义：  

![](https://i.imgur.com/QEjLRmC.png)  

结构体的构造函数：
![](https://i.imgur.com/895j95l.png)  

![](https://i.imgur.com/rj0dvWS.png)  

接下来还需要一个激活组件的函数：  

![](https://i.imgur.com/ldnMqml.png)  

4、结构体写好了，我们还需要到.h文件中定义所需的public函数和private变量：  

![](https://i.imgur.com/RPHpmF7.png)  

![](https://i.imgur.com/txBa6tA.png)  

然后到.cpp中实现函数：  

![](https://i.imgur.com/tKvihPo.png)  

![](https://i.imgur.com/Kc7oTgj.png)  

5、现在可以实现SubmitEvent函数了：  

![](https://i.imgur.com/zS0kw77.png)  

6、由于我们还要实现把输入框的文字也添加到ChatShow这个控件中，所以来到ChatRoomWidget.h中，定义一个委托：  

![](https://i.imgur.com/PAB2RR0.png)  

并在public下定义委托变量：  

![](https://i.imgur.com/c7KMsmr.png)  

然后回到.cpp文件中，在SubmitEvent函数最后执行委托：  

![](https://i.imgur.com/t8oZLXr.png)  


7、由于SendEvent是通过按钮发送信息，所以把SubmitEvent函数复制一下，粘贴到SendEvent函数中即可。只有return的部分不同：  

![](https://i.imgur.com/Cu5fzLu.png)  

8、再在.h中public下定义一个滑动到底部的函数，用在打开聊天室是执行：  

![](https://i.imgur.com/TlXgeuT.png)  

实现只有一条语句：  

![](https://i.imgur.com/PiYnVRV.png)  

9、我们来到GameHUDWidget.cpp，找到InitUIMap函数，把聊天室的委托绑定：  

![](https://i.imgur.com/VDh0ZxA.png)  

然后在Tick函数中让聊天室也每五秒插入一条信息：  

![](https://i.imgur.com/Ea755Vp.png)  

还要找到ShowGameUI函数，在切换UI的后面添加语句，让滑动框滚到最下面：  

![](https://i.imgur.com/yhUHwMH.png)  

10、打开虚幻编辑器，打开BPSlAiGameStyle，把ChatRoomBGBrush指定一下。这里不需要Image，直接修改颜色和透明度即可：  

![](https://i.imgur.com/HlggIGN.png)  

保存一下，运行游戏，按T键打开聊天室，输入信息，按Enter键或者点击发送，都可以发送信息：  

![](https://i.imgur.com/qg3rEeX.png)  

