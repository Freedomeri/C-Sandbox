#第七节 进入游戏控件和菜单控件初始化的实现     
    
##7.1 进入游戏控件    
1、**制作控件显示部分**    
（1）现在我们要开始制作进入游戏控件，我们首先在UE4中新建这样两个蓝图：   
![](https://i.imgur.com/9cOwIf3.png)      
    
![](https://i.imgur.com/BoPsOHN.png)    
（2）然后打开VS，在SSlAiMenuWidget.cpp文件中添加以下两个头文件：   
![](https://i.imgur.com/2PrHJjH.png)    
并实例化新建的NewGameWidget:   
![](https://i.imgur.com/uMLuO8b.png)    
(3)接下来我们修改SSlAiNewGameWidget.h和SSlAiNewGameWidget.cpp内容如下：   
             
SSlAiNewGameWidget.h         
![](https://i.imgur.com/5Q94Ohj.png)       
      
SSlAiNewGameWidget.cpp      
![](https://i.imgur.com/84WwAZp.png)     
![](https://i.imgur.com/IkSsP0d.png)     
![](https://i.imgur.com/flgX2LN.png)   
     
HintText的功能是在为输入内容是显示提示信息，就比如在输入密码的框内显示“请输入密码”。   
这样子实现进入游戏控件的显示部分就做完了。    
2、**制作控件功能部分**    
（1）这里我们要做的是获取输入的存档名，并创建一个存档。所以我们先在SlAiDataHandle.h下定义一个存档名:   
![](https://i.imgur.com/XEL2Yn2.png)    
再在SlAiDataHandle下初始化一下RecordName并定义为空:   
![](https://i.imgur.com/0E0I66J.png)     
然后我们需要写一个外部调用的方法来通过存档进入游戏，并检查时候是重复的存档名：修改SSlAiNewGameWidget.h如下：   
![](https://i.imgur.com/3XG3y18.png)     
     
修改SSlAiNewGameWidget.cpp如下：      
![](https://i.imgur.com/7NquVhG.png)         
![](https://i.imgur.com/ywfhO5v.png)     
![](https://i.imgur.com/klVW8jJ.png)    
![](https://i.imgur.com/ldAPy4y.png)    
再将SSlAiMenuWidget.cpp下的存档改成SSlAiChooseRecordWidget：   
![](https://i.imgur.com/jS8teEW.png)    
现在我们就可以去看一下实际效果了   

（2）接下来我们要做下拉菜单，这个的样式和之前的输入存档差不多，我们可以直接复制粘贴，最后修改一下最后的EditTextBox为RecordComboBox,因为我们要做的是点击出现下拉菜单，选择我们的存档，开始游戏。   
先设置SSlAiChooseRecrodWidget.h   
![](https://i.imgur.com/mWi6kP0.png)    
    
再设置SSlAiChooseRecrodWidget.cpp,这里我们把SSlAiNewGameWidget.cpp中的样式黏贴了过来，只修改了最后一部分，所以不把未修改的贴出来了。   
![](https://i.imgur.com/fCuZYvd.png)     
这里我们用循环，把本来保存再RecordDataList中的存档数据转换成FString类型再保存到OptionSource中。    
       
![](https://i.imgur.com/FOQb9ym.png)    
![](https://i.imgur.com/xq3Koym.png)   
   
最后让我们来看一下最终的效果图：   
![](https://i.imgur.com/7dHCx7K.png)        
                 
##7.2 菜单控件初始化   
1、*菜单界面实现***    
（1）这一节我们要制作剩下所有的菜单界面。我们主要需要修改的是SSlAiMenuWidget下的ContentBox下的内容。我们先到上面添加一个结构体，每个结构体对应这一个菜单。    
![](https://i.imgur.com/uZPHZPq.png)    
然后添加SSlAiMenuWidget.h内容如下：    
![](https://i.imgur.com/um7VCgN.png)    
![](https://i.imgur.com/I1D9vld.png)    
这里我们新建了三个指针来保存3个游戏菜单的标题，来修改现在的Menu，还新建了一个初始化所有控件的函数，来方便使用。现在我们要做的就是将他们全部实例化，并保存在MenuMap里。我们先添加SSlAiMenuWidget.h代码如下：   
![](https://i.imgur.com/BUjXDqu.png)    
   
然后再修改SSlAiMenuWidget.cpp文件代码如下：   
![](https://i.imgur.com/zljKa6R.png)      
这里我们把原先写的换成了初始化界面，但我们还没有实现这个函数，所以我们接着写：   
![](https://i.imgur.com/yJL1cOh.png)    
![](https://i.imgur.com/msVLdy3.png)     
![](https://i.imgur.com/koZl65h.png)   
![](https://i.imgur.com/6X5cS3g.png)    
这里我们写了5个界面的按钮。   
2、**调用这些函数**   
（1）添加SSlAiMenuWidget.h代码如下：   
![](https://i.imgur.com/y4AUJWh.png)    
![](https://i.imgur.com/1Sk1oM7.png)   
          
再添加SSlAiMenuWidget.cpp代码如下：    
    
![](https://i.imgur.com/EH8U3US.png)    
![](https://i.imgur.com/FlTRmP3.png)     
            
在运行之后我们就会将所又的菜单实例化，然后调用Choosewidget来调用第一个Menu为Mainmenu。再将Childwidget中的所有组件添加到ContentBox中，再修改标题和Size。   
现在我们来看一下运行效果：   
![](https://i.imgur.com/dIpcYs6.png)   
