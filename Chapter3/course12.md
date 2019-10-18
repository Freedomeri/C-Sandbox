#第12节 制作快捷栏UI、物品属性Json读取  

##12.1 制作快捷栏UI  

要想实现按照手里拿的武器播放过不同动画，需要制作一个物品栏的UI界面。  

1、打开虚幻编辑器，新建一个用于游戏UI的Style样式（继承SlateWidgetStyle）：  

![](https://i.imgur.com/TWMX22L.png)  

然后开始建UI控件，先创建一个在HUD下的根控件：  

![](https://i.imgur.com/hwua2fX.png)  

继续创建一个快捷栏控件：  

![](https://i.imgur.com/TwhcXBL.png)  

2、打开VS工程，首先要将GameHUDWidget根控件添加到GameHUD中。先在SlAiGameHUD.h下声明构造函数和指针，并在.cpp文件中实现：  

![](https://i.imgur.com/v36NN6m.png)  

![](https://i.imgur.com/krGNiJt.png)  

通过自动提示添加的头文件不够，需要再手动添加SlateBasics.h  

![](https://i.imgur.com/pXmb84R.png)  

3、下一步需要将快捷栏添加到根HUD控件中。先在GameHUDWidget.h中定义所需变量和函数，仿照制作Menu菜单时的过程：  

![](https://i.imgur.com/8Jbogy4.png)    

然后在.cpp中实现两个函数：  

![](https://i.imgur.com/PgGW1cJ.png)  

之后就可以在Construct函数中写UI内容了：  

![](https://i.imgur.com/eXK2Z89.png)  
到这里我们的GameHUDWidget算是写完了。

4、接下来是写ShortcutWidget快捷栏UI，为方便理解，先看一下要做的效果：  
![](https://i.imgur.com/96zu1pJ.png)  
与C++对应的蓝图布局：  

![](https://i.imgur.com/6py9ENz.png)  


首先在ShortcutWidget.h中定义GameStyle：  

![](https://i.imgur.com/6F2f9ZC.png)  

然后到.cpp文件中获取：  

![](https://i.imgur.com/tTyNDro.png)   
不要忘记添加我们的两个Style头文件。  

开始写UI控件框架：  

![](https://i.imgur.com/qzRudr6.png)  
其中的ShortcutInfoTextBlock是快捷栏文字的指针，我们需要先在.h文件中定义一下(private)：  


![](https://i.imgur.com/ifJuKhD.png)  

然后由于需要设置文字相关的样式，我们先去GameWidgetStyle.h中定义字体属性：  

![](https://i.imgur.com/My27v6N.png)  

![](https://i.imgur.com/3fZeYK3.png)  

![](https://i.imgur.com/HeWGdaS.png)  

![](https://i.imgur.com/kjzIEsm.png)  

为节省时间，可从原工程里面复制。  

定义完之后，回到ShortcutWidget.cpp文件，将字体属性设置好：  

![](https://i.imgur.com/8yJ3q6W.png)  

5、接下来是字体下面的表格UI：  

![](https://i.imgur.com/u2TSVWo.png)  
其中的GridPanel是表格控件的指针，我们同样需要先在.h文件中定义：  

![](https://i.imgur.com/B5gRK0S.png)  

表格下面的控件比较多，比较麻烦，我们一会儿动态添加。  


6、在ShortcutWidget.h中定义私有初始化容器的方法，来将一个个的物品栏方块初始化：  

![](https://i.imgur.com/5y7yo13.png)   

然后到.cpp文件中实现：  
![](https://i.imgur.com/R6duWPs.png)  
由于每个物品的信息我们都要保存下来，所以在这里定义三个指针，一个是最外边的背景框，一个是里面的物品图标，一个是物品的数量文字。  

继续添加代码：  
![](https://i.imgur.com/FmEWQxY.png)  

最后要将这些容器添加到表格控件中，所以继续在这个函数中的for语句中添加：  


![](https://i.imgur.com/LBSlbdE.png)  
每创建一个容器，就将它添加到表格中，第i列第0行。  

7、初始化容器函数写好之后，我们要把他放在Tick函数中运行。来到.h文件，先定义一个私有bool变量：  

![](https://i.imgur.com/0EXVZ9J.png)  

然后到public下重写Tick函数：  

![](https://i.imgur.com/Kppt79l.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/Rp1RRj7.png)  

同时在Construct函数中设置bool值变量为false：  

![](https://i.imgur.com/Y5sTyNQ.png)  

8、开始指定容器内容的资源文件。首先去到GameWidgetStyle.h中定义笔刷：  

![](https://i.imgur.com/kVSXdt2.png)  
  

一个是正常状态的背景图片，一个是被选中时的图片，最后一个是没有物品时的背景。  
  

回到ShortcutWidget.cpp中，设置图片：   

![](https://i.imgur.com/PNFQsqT.png)  

设置这些暂时是为了看效果，之后会用设置其他代码。  

9、启动项目，打开虚幻编辑器。在Content的UI/Style文件夹下创建Slate Widget Style，继承我们的SlAiGameStyle类，命名为BPSlAiGameStyle。  

![](https://i.imgur.com/nJCTwc5.png)  

打开此Style，依次设置资源文件：  
Normal Container Brush：  

![](https://i.imgur.com/d18LLFI.png)  

Choosed Container Brush：  

![](https://i.imgur.com/u3wYIbW.png)  

Empty Brush不用设置图片，把Draw As属性设置为None就可以了。  

下面Common标签下的字体属性：  

Font 60：  

![](https://i.imgur.com/GUjHCvl.png)  

Font Outline 50：  

![](https://i.imgur.com/RgnJpCG.png)  
带有Outline的字体，就把Outline属性的轮廓大小调为3，颜色调为白色。  
后面的字体请自行设置。  

颜色：  

![](https://i.imgur.com/ofYba02.png)  

最后点击Save，运行游戏，可以看到效果：  

![](https://i.imgur.com/2E7ojej.png)  
  
  


##12.2 物品属性Json读取  

1、首先为了方便理解，我们可以打开工程项目的Content/Res/ConfigData目录，打开ObjectAttribute这个Json文件，观察一下里面的数据类型都有哪些。  


2、打开VS工程，先将上一小节的那些待删除代码删掉，暂时用不到了。  

去到SlAiTypes.h，定义一个物品类型的枚举：  

![](https://i.imgur.com/xpbfG0X.png)  

接着在下面定义一个物品属性的结构体：  

![](https://i.imgur.com/XwT4rD4.png)  
  

![](https://i.imgur.com/SsJRtou.png)  

  
3、在SlAiJsonHandle.h文件中定义所需变量(private)：  

![](https://i.imgur.com/0PKmPhA.png)  


然后在public下创建一个Json解析函数：  

![](https://i.imgur.com/Gubbiqu.png)  

在实现之前，先到.cpp文件的构造函数中将文件名赋值：  

![](https://i.imgur.com/NgQMOup.png)  

然后再实现Json解析函数，具体模式与解析Menu时基本相同：  

![](https://i.imgur.com/eeIvuCd.png)  

![](https://i.imgur.com/qomLPAa.png)  

其中，获取到的ObjectType是字符串类型，然而我们自己定义的是枚举类型，所以我们还需要写一个转换方法（由于我们的枚举没有定义在UENUM下面，所以不能用之前写的GetEnumFromString方法）：  

在.h文件中定义一个private方法：  

![](https://i.imgur.com/1ef4AUe.png)  

然后到.cpp文件实现：  

![](https://i.imgur.com/It8fAGc.png)  


有了这个转换函数，我们就可以继续在上面函数的for语句中继续添加代码：  

![](https://i.imgur.com/VlbZEdi.png)  


然后在if语句后加一个解析失败的debug：  

![](https://i.imgur.com/6FeVWKj.png)  
读取的方法就完成了。  

4、我们打算在DataHandle中调用解析函数。先在SlAiDataHandle.h中定义public变量：  

![](https://i.imgur.com/1GqQRR6.png)  

由于在菜单界面的时候，DataHandle已经被实例化，我们不能再将物品读取的方法写在构造函数中了。我们需要定义方法，让读取物品数据的方法在跳转场景时运行（后面会写在SlAiGameMode中）：  

首先在public下定义游戏数据初始化的方法：  

![](https://i.imgur.com/MUezucN.png)  

然后在private下定义物品数据初始化的方法：  

![](https://i.imgur.com/IfNxm1n.png)  

去到.cpp文件实现，首先把物品初始化方法放到初始化游戏数据中，之后我们还有资源数据等，都会统一在游戏数据方法中调用：  

![](https://i.imgur.com/GK2ctZ4.png)  

然后在物品方法中实现初始化：  

![](https://i.imgur.com/3jYe8nJ.png)  

最后，我们在SlAiGameMode.cpp的BeginPlay中调用：  

![](https://i.imgur.com/28Htngx.png) 


5、写完这些之后，我们想要看到它是否能读取成功，需要在物品数据读取的方法中Debug一下。在DataHandle.cpp中添加：  

![](https://i.imgur.com/Oigb2U5.png)  

其中的ToString方法，是我们在SlAiTypes.h中写的一个函数，帮助输出Debug字符串。写在ObjectAttribute结构体里面：  

![](https://i.imgur.com/zlEmhxV.png)  

到这里我们就可以打开编辑器运行了。游戏中效果如下：  

![](https://i.imgur.com/RVgDkti.png)  

在保证物品数据读取没有问题后，我们可以先把Debug那条语句删掉了。  


6、下一步，我们要根据物品数据来设置快捷栏的内容。首先在SlAiTypes.h中定义一个快捷栏容器的结构体：  

![](https://i.imgur.com/PkAjzcU.png)  
  
记得添加控件的头文件。  


然后再在结构体里面定义初始化的构造函数：  

![](https://i.imgur.com/SgB0gqm.png)  

![](https://i.imgur.com/d7wK4y4.png)  

还有一些选择物品时相关的函数：  

![](https://i.imgur.com/vAGSuWp.png)  

![](https://i.imgur.com/dua2Xrj.png)  

![](https://i.imgur.com/JtTo29S.png)  

快捷栏的结构体就写好了。下一节我们就会使用它。  










  


