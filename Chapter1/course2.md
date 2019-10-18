#第二节 添加Slate界面并设置样式  、



  

##2.1添加Slate到界面  
1、**设置默认关卡**。  
在新建的工程中，在 **Content** 目录下新建Maps文件夹，并在Maps下新建一个Level，命名为 **MenuMap** 。打开 **ProjectSettings** ，把新建的关卡设为默认关卡。  

![](https://i.imgur.com/WIrplhI.png)  

![](https://i.imgur.com/JUluHT2.png)  
2、**搭建GamePlay框架**。  
在左侧内容浏览器中添加三个新C++类。分别继承 **GameModeBase** 类、 **PlayerController** 类、 **HUD** 类。


![](https://i.imgur.com/pj9S6vv.png)  

![](https://i.imgur.com/vJpcysw.png)  

  

创建好基础类之后，要记得在ProjectSettings里面把DefaultGameMode切换成我们刚刚创建的GameMode。

（注：命名时最好以工程名的大写字母开头。）  

3、**添加Slate模块**。  
在打开的VS工程中打开 **“工程名.Build.cs”** 文件，在其中添加"Slate"、"SlateCore"模块。  
![](https://i.imgur.com/J8uLSy1.png)  

4、**显示鼠标**。   
设置默认Controller和HUD： 
![](https://i.imgur.com/sShPr3T.png)  
SlAiMenuGameMode.h  

将鼠标放在.h文件的构造函数上，VAssistX插件会在下方显示一个小箭头。通过点击可以快速创建.cpp的定义。

![](https://i.imgur.com/IPPDWrB.png)  
SlAiMenuGameMode.cpp  

![](https://i.imgur.com/HBMmtKx.png)  
SlAiMenuController.h  

锁定鼠标在窗口之内：  
![](https://i.imgur.com/GyZ3mSL.png) 
SlAiMenuController.cpp 


5、**添加Slate**。  
在引擎内容浏览器中创建两个类，继承 **Slate** 类，分别命名SBMenuWidget和SBMenuHUDWidget（这个将是主Widget）。目录结构如下图：  
![](https://i.imgur.com/1Fqb7HD.png)  

向屏幕添加HUDWidget  
![](https://i.imgur.com/IctefnG.png)  
SlAiMenuHUD.h  

![](https://i.imgur.com/tGf1DqS.png)  
SlAiMenuHUD.cpp  

在MenuHUDWidget构建的时候创建按钮  
![](https://i.imgur.com/r10tI54.png)  
SSlAiMenuHUDWidget.cpp  

此时在VS中启动调试可以看到一个充满屏幕的按钮。  

##2.2使用WidgetStyle设置样式  

1、创建一个继承SlateWidgetStyle的样式类：SlAiMenuWidgetStyle。  
![](https://i.imgur.com/1xsiMGc.png)  

2、再创建一个不继承任何类的SlAiStyle类，用来获取我们的WidgetStyle。  
![](https://i.imgur.com/bMVnR1L.png)  

3、在SlAiStyle.h中写添加所需函数:   
![](https://i.imgur.com/vmM5DbU.png)   
Initialze函数用来判断单例是否可用，若不可用则通过Create方法创建此单例。

4、在SlAiStyle.cpp中实现这些函数：  
![](https://i.imgur.com/evGxeKu.png)  

其中Initialze函数实现样式的初始化。若样式实例不可用，则重新创建并注册样式。  

![](https://i.imgur.com/yJudLNI.png)  

ShutDown中取消注册样式，并确定样式实例是否唯一。最后重设样式实例。  
Create()方法在游戏的所示路径下获取样式的资源。  

5、为防止样式实例为空，要在"SlAiCourse.cpp"中调用初始化函数。Source文件夹下的SlAiCourse文件夹可以看成是一个模组"Module"，项目启动时会加载这个模组文件。  

![](https://i.imgur.com/hOf68gb.png)  
SlAiCourse.h  

![](https://i.imgur.com/ca7BwlP.png)  
SlAiCourse.cpp  

6、定义样式  
在SlAiMenuWidgetStyle.h中添加笔刷定义  

![](https://i.imgur.com/WhaF5Pe.png)  

在SSlAiMenuHUDWidget.h中添加Menu样式指针的定义（指向之后我们在编辑器蓝图里继承此样式类的资源）  
![](https://i.imgur.com/Vpd8OKL.png)  

7、获取到样式并通过笔刷添加背景图片  
在SSlAiMenuHUDWidget.cpp中添加以下代码：
![](https://i.imgur.com/9ft65O4.png)  

8、在Content目录下新建/UI/Style目录，在其中创建一个继承自SlAiMenuWidgetStyle的SlateWidgetStyle，命名为上图所示的"BPSlAiMenuStyle"。  

![](https://i.imgur.com/YhHIuLx.png)  

打开这个Style，将Image属性设置为BGImage，点击Save保存。  

点击Play，可以看到下图效果：  

![](https://i.imgur.com/PvJpJ91.png)  

----------




注：关于TSharedPtr，虚幻智能指针，可参考官方文档：  
[http://api.unrealengine.com/latest/CHN/Programming/UnrealArchitecture/SmartPointerLibrary/index.html](http://api.unrealengine.com/latest/CHN/Programming/UnrealArchitecture/SmartPointerLibrary/index.html)
