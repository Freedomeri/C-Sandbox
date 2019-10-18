#第三节 DPI屏幕配适和菜单布局实现        
       
##3.1 DPI屏幕配适
1、**了解界面适配**    
（1）SOverlay控件常用于UI布局，下面我们在SSlAiMenuHUDWidget.cpp中实现这些函数:   
    
![](https://i.imgur.com/8oLAUnN.png)   
     
SOverlay类和蓝图类是一一对应的关系，也就是说，用Slate Widget Style 的SOverlay类下的每一个函数都对应这蓝图类Widget Blueprint下的参数设定。下面我们就再做一份Widget Blueprint来实现一边上面的代码以方便你们理解。   
（2）我们先创建一份Widget Blueprint类，打开，先删掉NewWidgetBlueprint下的Canvas Panel，然后在NewWidgetBlueprint下添加Overlay。这一步就相当于Slate Widget Style中的SNew(SOverlay)。现在我们再在Overlay下添加Image来添加图片，同理这一步就是SNew(SImage)。接下来我们在蓝图中将Image->Detail->Horizontal Alignment 和Vertical Alignment都设置为最后一项，意思是左右，上下填满。这一步就对应了Slate Widget Style中的.HAlign(HAlign_Fill)和.VAlign(VAlign_Fill)。最后我们在Appearance->Brush->Image下添加图片BGImage。就完成了之前在Slate Widget Style中做的工作。   
![](https://i.imgur.com/l8i5jwb.png)            
（3）但是这样一个上下左右都拉满的图片不方便我们研究界面适配，所以我们在插入一张图片。   
![](https://i.imgur.com/60xgKjX.png) 
       
在Slate Widget Style中写完后，再在BPSlAiMenuStyle添加图片，来绑定。     
![](https://i.imgur.com/fUIT0sB.png)    
现在我们运行程序，会有如下画面：   
![](https://i.imgur.com/yXI3E5J.png)   
因为之前的设置，所以我们还不能对这个窗口进行缩放，所以我们要对SlAiMenuController.cpp进行如下更改：   
![](https://i.imgur.com/E8kdYjv.png)    
现在我们再来看一下缩放的效果，我们会发现中间的图标不是随着拉动线性缩放的。    
![](https://i.imgur.com/Crjt048.png)    
2、**自定义界面适配**    
（1）我们可以在ProjectSetting->Engine->User Interface->DPI Scaling中对界面配适做出更改，这里我们使用自定义配适，将DPI Scale Rule 改为Custom。写下来我们就该手写这个界面适配了。   
修改SSlAiMenuHUDWidget.h代码如下：  
![](https://i.imgur.com/0CdqwkA.png)     
然后在SSlAiMenuHUDWidget.cpp添加代码如下：   
![](https://i.imgur.com/i6ri7WJ.png)   
![](https://i.imgur.com/vHzDFAb.png)  
![](https://i.imgur.com/y8LL0Ey.png)   
我们用GetViewportSize来获取屏幕的尺寸，这里用的电脑是1920x1080的，然后用GetUIScaler获取界面和屏幕的比例，再在开头调用Bind赋值给UIScaler，最后SNew一个SDPIScaler来改变长宽。   
运行的效果如下：   
![](https://i.imgur.com/F88cBVJ.png)   
（2）最后我们将鼠标再次锁定在界面里：   
![](https://i.imgur.com/0U558Y7.png)   
    
##3.2菜单布局实现   
1、**实例化MenuWidget**      
（1）我们先编写SSlAiMenuHUDWidget.h如下：   
![](https://i.imgur.com/UdqeAVD.png)    
然后我们再编写SSlAiMenuHUDWidget.cpp如下：   
![](https://i.imgur.com/YWgrEB6.png)    
这里我们用SAssignNew(MenuWidget,SSlAiMenuWidget)将MenuWidget实例化，但是他还什么内容都没有。然后我们就暂时用不到这两个文件了。   
 
2、**实现菜单布局**   
（1）现在我们来编写SSlAiMenuWidget.h：   
![](https://i.imgur.com/JYADuXh.png)    
![](https://i.imgur.com/8qYkvkX.png)    
如果这里嫌麻烦，可以把MenuStyle封装到SlAiStyle中。SBox就相当于UI界面的Size Box,这个组件可以定义Width Override和Height Override。另外SBox只能插入一个子主键。接下来我们修改SSlAiMenuWidget.cpp内容，你就会看到，但我们先修改SlAiMenuWidgetStyle.h如下：   
![](https://i.imgur.com/9vsncWa.png)   
![](https://i.imgur.com/XGYRanL.png)   
![](https://i.imgur.com/xRkgVFZ.png)   
这里我们定义了背景图片，左右图标和标题。现在我们就可以去编写SSlAiMenuWidget.cpp的内容了：    
![](https://i.imgur.com/jaYJgQc.png)   
![](https://i.imgur.com/tUkIoub.png)    
![](https://i.imgur.com/vrIBjcO.png)   
![](https://i.imgur.com/yhFvrmP.png)     
我们这里定义了他们在SBox中各自的位置。并在标题中用STextBlock设置了文字标题，这里我们先都用英文，因为中文会显示乱码，或者报错，这个问题之后我们会解决。现在我们打开UE，编辑BPSlAiMenuStyle,给之前设定的两个图标和一个标题绑定图片：   
![](https://i.imgur.com/gNMhYvN.png)   
这样我们就完成了，现在我们看看效果图片：   
![](https://i.imgur.com/3WUMIPG.png)   
