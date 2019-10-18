#第21节 OnPaint实现背包拖拽；丢弃物品与绑定快捷栏  

##21.1 OnPaint实现背包拖拽   

1、首先我们在PackageWidget.h文件中重写public绘制函数：  

![](https://i.imgur.com/vXx7b53.png)  

然后在.cpp文件中先调用父类函数：  

![](https://i.imgur.com/HdV1hqE.png)  

2、先来到PackageManager.h中，定义两个Public变量：  

![](https://i.imgur.com/MTnMs93.png)  

并将它们初始化：  

![](https://i.imgur.com/SOB3PTX.png)  
在这里为了方便调试看效果，我们先不设为0。  

3、接下来回到PackageWidget.cpp中，继续写OnPaint函数：  

![](https://i.imgur.com/HAexSmv.png)  

利用MakeBox函数，渲染一个Box在鼠标的位置。第二个参数是Layer的优先级，优先级越高，在屏幕显示的越靠前。第四个参数通过我们早先在DataHandle里面定义的笔刷列表，获取到当前物品ID的图片笔刷。  

![](https://i.imgur.com/jhPf04n.png)  

由于工具和武器不允许叠加，所以不渲染它们的文字。  

最后在OnPaint函数最后返回值：  

![](https://i.imgur.com/Wu9E0VQ.png)  

现在编译打开虚幻编辑器，运行游戏，按E键打开背包，可以看到一个苹果（ObjectIndex为3）附着在鼠标指针上了。  

4、实现点击背包将物品放进去。  
先在PackageManager.h中定义两个public函数，供PackageWidget调用：  

![](https://i.imgur.com/W9XWk2C.png)  

然后去到PackageWidget.h中，重写鼠标点击事件：  

![](https://i.imgur.com/8BvszgF.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/VCNJ7Bo.png)  

![](https://i.imgur.com/mosSZj0.png)  

5、至于LeftOption的函数实现，在我们把Container的动态笔刷写好再去实现。  
先来到ContainerBaseWidget.h下面，定义Protected变量：  

![](https://i.imgur.com/XzIq3V3.png)  

然后在.cpp构造函数中初始化：  

![](https://i.imgur.com/FoFdIsj.png)  

回到.h文件中，定义public函数：  

![](https://i.imgur.com/YlUyutn.png)  

并在.cpp中实现：  

![](https://i.imgur.com/U98ajDF.png)  

6、除了判断ID之外，还要判断物品是否可以叠加，显示数量。需要继续在ContainerBaseWidget.h中添加protected函数：  

![](https://i.imgur.com/JGZMxAo.png)  

并在.cpp中实现：  

![](https://i.imgur.com/SoYTX11.png)  

接下来就把这个函数应用到ResetContainerPara函数中继续判断。在ResetContainerPara函数中添加以下代码：  

![](https://i.imgur.com/AE4af4C.png)  

7、在.h文件中定义两个函数用来获取ID和Num，以提供给其他的类使用：  

![](https://i.imgur.com/uQ8zQdX.png)  

并在.cpp中直接return：  

![](https://i.imgur.com/ORWYtAU.png)  


8、下面要实现鼠标点击把物品放到物品栏中，并更新状态。内容比较难理解，建议自己反复琢磨。  

首先在ContainerBaseWidget.h中定义两个public函数：  

![](https://i.imgur.com/I0lJxlp.png)  
函数前两个参数是鼠标当前物品的ID和数量；后两个参数是在点击事件之后，鼠标上还应该存在的物品ID和数量。  

在游戏中的实际逻辑建议自己运行一下源程序，体会一下背包左右键的作用。  

在.cpp文件中，先实现LeftOperate的第一部分：  

![](https://i.imgur.com/h5Qf2Nl.png)  

然后是第二部分，鼠标物品与物品栏物品不相同：  

![](https://i.imgur.com/j1Yxawi.png)  

左键的逻辑就写完了。  

9、接下来是右键的逻辑。在RightOperate函数中添加代码：  

![](https://i.imgur.com/Mqmdxro.png)  

第二种情况：  

![](https://i.imgur.com/hqUj0nE.png)  

第三种情况：  

![](https://i.imgur.com/8EFvxxH.png)  

这样左右键的逻辑就已经完善了。   

10、回到PackageManager.cpp中，实现LeftOption和RightOption：  

LeftOption：  

![](https://i.imgur.com/dCqaD5F.png)  

RightOption：  

![](https://i.imgur.com/le8EQCz.png)  

11、运行游戏，由于物品栏暂时是空的，暂时先拿手上的苹果简单测试左右键的功能。  






##21.2 丢弃物品与绑定快捷栏  

1、先在ContainerBaseWidget.h中定义几个会用到的委托：  

![](https://i.imgur.com/LOIHSEA.png)  

第一个委托是在往合成框里面放物品的时候，会提前在输出框中看到可以合成出什么物品。第二个委托是提取输出框物品的时候，处理输入框和输出框物品的信息，并更新。  

然后将这几个委托在public下声明：  

![](https://i.imgur.com/I3LBuWs.png)  

2、来到ContainerOutputWidget.h下，重写两个public方法：  

![](https://i.imgur.com/rRfLOcE.png)  

然后到.cpp文件中先实现左键操作：  

![](https://i.imgur.com/Mzww0lL.png)  

![](https://i.imgur.com/02ImM57.png)  

![](https://i.imgur.com/kRZMxzG.png)  

然后是右键操作：  

![](https://i.imgur.com/fwTllim.png)  

![](https://i.imgur.com/rB6NnlX.png)  

![](https://i.imgur.com/gi0cdqo.png)  

![](https://i.imgur.com/ehCl7w0.png)  

算法中具体每一种情况，可以打开源程序自己试验一下。  

3、输出框的操作写完了，接下来是输入框。打开ContainerInputWidget.h，重写父类的方法：  

![](https://i.imgur.com/Y5wY6cj.png)  

在.cpp中实现：  

![](https://i.imgur.com/LwcHgjd.png)  

![](https://i.imgur.com/92P73LG.png)  

每当合成输入框改变的时候，就会调用CompoundInput委托，委托的内容我们之后会写，是把输入框遍历一遍，看是否可以合成物品，并修改输出框的Image。  


4、然后是快捷栏。打开ContainerShortcutWidget.h，和输出框一样重写父类方法，并在.cpp中实现：  

![](https://i.imgur.com/N1Ws183.png)  

![](https://i.imgur.com/Filg9IM.png)  

5、控件中的事件都写完了，接下来去到PackageManager.h中，在private下定义四个函数，对应四个委托的绑定：  

![](https://i.imgur.com/jAS4uRH.png)  

在实现之前，先把委托绑定上。到PackageManager.cpp中，找到InsertContainer函数，每添加一个容器，就给它绑定相应的委托：  

![](https://i.imgur.com/s3MmWWb.png)  

![](https://i.imgur.com/m81oDZs.png)  

因为Normal类型的正常背包不需要特殊的操作，所以我们不用绑定委托，基类的函数已经足够实现它的功能了。  


6、在PackageManager.h下声明两个public的ContainerBaseWidget的委托，供事件调用：  

![](https://i.imgur.com/2sgLq7v.png)  

然后就可以在刚刚定义的四个函数中实现了。来到.cpp文件：

![](https://i.imgur.com/LpQnJrR.png)  

![](https://i.imgur.com/IDWZNil.png)  

7、在绑定委托之前，先实现需要绑定的函数。来到PlayerCharacter.h，在public下定义函数：  

![](https://i.imgur.com/3fAHnHb.png)  

并在.cpp中实现：  

![](https://i.imgur.com/TuihXOG.png)  

8、先实现一部分，丢弃物品我们还需要一个物品往前扔的效果，所以要在FLobObject中写一个初始化丢弃物品的方法：  
去到FLobObject.h，在public下定义

![](https://i.imgur.com/NINIj8k.png)  

然后在.cpp中实现：  

![](https://i.imgur.com/oMVwC4n.png)  

![](https://i.imgur.com/E1sNLFX.png)  

9、实现之后，再回到PlayerCharacter.cpp中，将没有写完的丢弃函数完善：  

![](https://i.imgur.com/FMJmjXf.png)  

10、最后绑定委托。在SlAiGameMode.cpp中的InitializePackage函数下添加：  

![](https://i.imgur.com/1yucpuO.png)  

然后把PackageManager中丢弃物品的函数添加到LeftOption函数中：  

![](https://i.imgur.com/va1aLrK.png)  

运行游戏，把物品放在空白区域点击左键，物品已经可以丢弃了。  

11、接下来是更改快捷栏。先在PlayerState.h中定义public方法：  

![](https://i.imgur.com/S063Mx9.png)  
参数分别是快捷栏的ID、物品的ID以及物品数量。  

由于实现的时候需要用到控制器PlayerController，所以先在头文件public下定义控制器指针：  

![](https://i.imgur.com/9OWJkqJ.png)  

然后在protected下重写BeginPlay函数，并在BeginPlay中给Controller赋值：  

![](https://i.imgur.com/dmoiydi.png)  

现在可以继续实现ChangeHandObject方法了：  

![](https://i.imgur.com/6iM11cB.png)  

12、最后到GameMode中将更改快捷栏的方法也绑定到委托：  

![](https://i.imgur.com/IHMwurv.png)  

运行游戏，打开背包，把苹果放到底部的快捷栏的位置，可以看到游戏界面的快捷栏也同步物品了。而如果放的快捷栏位置是当前选中的位置，还可以看到手持的物品也会同步。  





