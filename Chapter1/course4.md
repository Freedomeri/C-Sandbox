#第四节 语言切换功能实现及自定义Button控件  

##4.1实现语言切换功能  

1、**让中文可以显示**  
（1）点击VS中的“文件->高级保存选项”，将编码设置为Unicode（UTF-8）。若无此菜单，可在“工具->自定义->命令”下自行添加。  
2、**创建本地化文字。**  

（1）首先创建一个SlAiInternation空白类，放在Data文件夹下。在其中实现UI上文字的本地化。  
![](https://i.imgur.com/G4nZg5A.png)  

删掉SlAiInternation的.cpp文件，只需要.h文件。  

在SlAiInternation类里面定义注册函数（其实是为方便创建本地化文字）  

![](https://i.imgur.com/EN5vpSv.png)  

在类之外通过本地化文字命名空间的方式创建  
![](https://i.imgur.com/7TX04cP.png)  

然后在SSlAiMenuWidget.cpp里面

![](https://i.imgur.com/B1knrlc.png)  

（2）将本地化英文转为中文。  
打开虚幻编辑器，在Window工具栏下的Localization Dashboard菜单中，通过Search Text files来找到我们在文件中创建的本地化文字  

![](https://i.imgur.com/9J6Romf.png)  

在Cultures标签下添加Chinese语言，点击Gather Text寻找。找完点击Chinese右边的Edit translation，在其中自行添加英文的翻译。  
翻译完之后回到此菜单，点击Count Words，待编辑器找完所有文字后，点击Compile Words，让编译器编译出它自己的本地化文字格式即可（默认保存在了Content->Localization文件夹下）。  

3、**中英文切换**  

（1）新建两个c++类，一个存放基础数据结构，一个实现功能。  

![](https://i.imgur.com/CnRYRtl.png)  

![](https://i.imgur.com/gFudAab.png)  

由于只需要数据结构的.h文件，所以删除掉SlAiTypes.cpp文件。  

在SlAiTypes.h中定义中英文的枚举类  
![](https://i.imgur.com/zC0YAL4.png)  
  

在SlAiDataHandle.h中仿照Style实现单例的创建和操作：

![](https://i.imgur.com/gLS3M9R.png)  

共有变量CurrentCulture用来公开当前的语言。  

在SlAiDataHandle.cpp中实现：  
![](https://i.imgur.com/7sLh2VB.png)  

![](https://i.imgur.com/dnx61bM.png)  

内置的FInternationalization的SetCurrentCulture方法可以实现中英文的切换。   


##4.2自定义Button控件  

1、在SlAiTypes.h文件中定义菜单的按钮枚举类型：  
![](https://i.imgur.com/vRQ96Jd.png)  

2、创建盛放菜单按钮的控件C++类：  

![](https://i.imgur.com/Fbl7o58.png)  


3、先将此控件插入到菜单中：首先在SSlAiMenuWidget.h中定义一个垂直列表指针  
![](https://i.imgur.com/Uu8l8Jf.png)  

需要注意，在.h文件中定义控件指针时，要在上面声明一下所用到的类。  

![](https://i.imgur.com/2lc7ltP.png)  

然后在.cpp文件中将此垂直列表插入到菜单根组件中  
![](https://i.imgur.com/0rWteVv.png)  

将刚刚创建的菜单项类添加到它的子控件中：  

![](https://i.imgur.com/BfdKbhz.png)  


   
4、在MenuItemWidget.h中定义一些控件属性。这里通过Slate自带的宏来创建。宏中第一个参数是此属性或事件的类型，第二个参数是属性或事件名（我们自己定义）。  

![](https://i.imgur.com/Z6bTEO5.png)  

其中的事件宏EVENT，需要传入一个委托，我们在上面定义传入一个参数的委托：  
![](https://i.imgur.com/xywzOM0.png)  
自定义不传出参数的委托使用DECLARE _ DELEGATE _ OneParam()  
其中第一个参数是委托名，第二个参数是委托的传入参数类型(此例为一个输入参数)。  
此处的按钮委托，意义是在按钮控件中定义这个按下事件的委托，然后传给SSlAiMenuWidget（菜单控件）中让它处理。  


下一步，我们需要声明一个函数，用来执行按下的事件  
![](https://i.imgur.com/3yyGNHR.png)  


之后我们就可以在SSlAiMenuWidget.cpp下使用刚刚创建的属性和事件宏了  

![](https://i.imgur.com/4iKL99Q.png)

6、在SSlAiMenuItemWidget.h中，需要获取我们定义的委托。  

定义一个private委托变量，再定义一个按下按钮类型的变量  
![](https://i.imgur.com/ttBo0bB.png)  

然后在SSlAiMenuItemWidget.cpp文件中获取这两个变量  

![](https://i.imgur.com/d03ceLR.png)

在控件类中，Construct：控件构造时执行的函数，其参数中的InArgs 可以获取到Slate宏定义的信息。  
格式为"InArgs._"+变量名。  


ItemType由于直接获取到的是EMenuItem命名空间，需要.Get方法才能获取到Type枚举类。  

7、开始写菜单按钮控件之前，还是要先获取MenuStyle（上图已获取），方法与SSlAiMenuWidget中一样。之后每个Widget控件在使用时都要获取一次MenuStyle样式。  
下面就开始设计菜单按钮：  

![](https://i.imgur.com/JEfDPT4.png)  

到这一步后，需要设置菜单按钮的背景图片，此时需要先定义一个笔刷去到SlAiMenuWidgetStyle.h中添加：  
![](https://i.imgur.com/SzSo6sM.png)  

回到菜单按钮控件中，将图片笔刷设置好，再添加按钮文字：  
![](https://i.imgur.com/oy3Fm8T.png)  

文字是鼠标按下时，获取的当前控件的ItemText属性。这个属性就是我们之前定义的Slate宏。Font，字体大小属性，我们需要在SlAiMenuWigetStyle文件中定义：  

![](https://i.imgur.com/FYpkdiF.png)  

8、重写菜单按钮控件的鼠标事件：  

![](https://i.imgur.com/SE3E3Go.png)  

然后定义一个私有bool变量，用来标识按钮是否被按下。  

![](https://i.imgur.com/vQY3cvn.png)  

并在.cpp的Construct构造方法中赋初值：  

![](https://i.imgur.com/uMfPSfF.png)  

接下来将三个鼠标事件实现：  

![](https://i.imgur.com/xdc2k7B.png)  

![](https://i.imgur.com/vRgveMv.png)  

![](https://i.imgur.com/IRl6BIq.png)  

9、按下按钮时的效果，我们用改变按钮颜色的形式实现。  
首先定义一个获取颜色的私有函数：  

![](https://i.imgur.com/ibR8YKX.png)  

在.cpp文件中实现它：  
![](https://i.imgur.com/2ly8reL.png)  

当按下时，变为半透明的红色。  
然后在Image组件中，设置颜色：  

![](https://i.imgur.com/zviVX3u.png)  

最后，为了看到按钮按下执行了委托事件，在SSlAiMenuWidget.cpp中填写事件：  

![](https://i.imgur.com/5DQg95D.png)  


这样按钮的代码就写好了。  

10、打开虚幻编辑器，将定义的笔刷、字体大小等编辑器属性设置好：  

![](https://i.imgur.com/RmyMNNT.png)  

![](https://i.imgur.com/zwDdlbz.png)  

![](https://i.imgur.com/fu2JzQw.png)  

启动游戏，可以看到点击按钮后：  
![](https://i.imgur.com/aZEN5Rp.png)


注：在使用vs启动虚幻编辑器时，会报一些莫名其妙的错误，一般图示的红色下划线的错误可以忽略。  

![](https://i.imgur.com/RQacwd6.png)  
更详细的说明请访问[https://www.cnblogs.com/sevenyuan/p/5685140.html](https://www.cnblogs.com/sevenyuan/p/5685140.html)