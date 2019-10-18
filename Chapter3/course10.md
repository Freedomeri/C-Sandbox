#第十节 动作蓝图与玩家移动   
    
##10.1 动作蓝图   
1、**vs部分**    
（1）先简单讲一下碰撞有关的只是，如果场景中需要两个物体发生碰撞,我们只需要勾选Generate Overtap Events,然后Object Type下面的这么多选项，意思是说，我们现在是pawn，如果我们勾选了WorldStatic的Block,那么在pawn和WorldStatic发生碰撞是就会检测到，并触发碰撞事件，而ignore就是忽略交互和碰撞事件。
![](https://i.imgur.com/mH80s3y.png)    
但这样一个一个设置是比较麻烦的，我们可以打开Project Setting，里面有一个Collsion，我们可以在其中的Preset建立一个属性表。现在我们就来给玩家添加碰撞，我们先在Object Channels下点击New Object Channel，设置其Name为Player：    
![](https://i.imgur.com/rqdKDZD.png)     
然后再New一个Preset：   
![](https://i.imgur.com/9roF64Y.png)    
这样我们人物在运动的时候就不会穿模了。   
   
（2）现在我们回到VS设置他的碰撞属性，添加SlAiPlayerCharacter.cpp如下：   
![](https://i.imgur.com/3rBIBzg.png)
然后我们就回到角色的蓝图，将他的Collision Presets设置为刚做的PlayerProfile。接下来我们就要开始写任务动作了。打开VS,添加SlAiPlayerAnim.h代码如下：    
![](https://i.imgur.com/Udv1lbC.png)   
这里的EdiAnywhere表示可以再任何的放使用，BlueprintReadOnly表示蓝图只读。接下来我们把刚才那两个参数初始化一下。添加SlAiPlayerAnim.cpp代码如下：   
![](https://i.imgur.com/d1mmJyQ.png)    
然后我们先做第三人称，第三人称需要速度，上半身旋转，还有跳跃。添加SlAiThirdAnim.h代码如下：   
![](https://i.imgur.com/kVYPz0s.png)    
IsInAir判断是否再空中。我们再初始化一下USlAiThirdPlayerAnim,修改SlAiThirdAnim.cpp代码如下：   
![](https://i.imgur.com/IhJ8MDD.png)    
现在我们运行UE，进行动作蓝图的编辑。
   
2、**ue部分**    
（1）我们现在Blueprint下新建一个文件夹叫做Player：   
![](https://i.imgur.com/tkOiw6H.png)    
然后我们再新建两个动画蓝图：   
![](https://i.imgur.com/cYmABm5.png)   
然后选中他的父类SlAiThirdPlayerAnim还有他的骨骼Player_Skeleton:   
![](https://i.imgur.com/nqwokC8.png)    
给他命名为ThirdPlayer_Animation。打开他，新建一个New State Machine，我们把他命名为LocalMotion：   
![](https://i.imgur.com/YEUMmEb.png)   
双击打开，加一个State，将他命名为Move，点进这个Move，看到右下角的Asset Browser有一个Player_Move.我们将它移过去，现在我们就要用到我们在c++中定义的函数了Get Direction和Get Speed：   
![](https://i.imgur.com/7iC9Y4O.png)
我们在Move后面再添加一个State，并将其命名为JumpStart,双击点开。添加Player_JumpStart，再添加Get Speed：   
![](https://i.imgur.com/IwmLjpI.png)    
回到LocalMotion，点进双箭头，编辑发生动作的条件：   
![](https://i.imgur.com/dYAYY2m.png)   
现在我们就要用到在C++中创建的Is In Air:   
![](https://i.imgur.com/mPPAoQP.png)   
为了要循环这个动作，我们再加一个State，将其命名为JumpLoop:   
![](https://i.imgur.com/fjC1mlB.png)   
点进JumpLoop，添加Player_JumpLoop,添加Get Speed：    
![](https://i.imgur.com/BrgyPK4.png)    
同样进入双箭头添加条件，添加Time Remaining(ratio)(Player_JumpStart)：     
![](https://i.imgur.com/6xjCSXr.png)     
再添加一个JumpEnd的State：    
![](https://i.imgur.com/OgTKAgz.png)   
双击打开JumpEnd添加Player_JumpEnd，再添加Get Speed：   
![](https://i.imgur.com/ye1ORRE.png)
最后将JunpEnd和Move连接。   
![](https://i.imgur.com/EMBF8GQ.png)   
添加判断什么时候JumpEnd。    
![](https://i.imgur.com/pZ4mEwx.png)   
最后添加JumpEnd转回到Move的条件：    
![](https://i.imgur.com/Svo38Ra.png)     
现在我们就可以连接LocalMotion和Final Animation Pose了。   
（2）现在我们回到VS中，讲我们刚才创建的蓝图添加到工程中。我们先获取动作的蓝图，如果是复制路径的话最后一定要添加_C，这样才是获取蓝图类：    
![](https://i.imgur.com/Rqs0OF0.png)   
将这一段代码放在给Mesh添加骨骼模型后面。    
现在运行程序，就能看到动作已经绑定上去了。现在我们再制作第一人称。在Player下在添加一个动作蓝图：     
![](https://i.imgur.com/lIyAd7c.png)   
![](https://i.imgur.com/uip4mhn.png)   
将其命名为FirstPlayer_Animation。打开他，新建一个New State Machine：    
![](https://i.imgur.com/zMV1xOV.png)    
点进去，只要设置一个FirstPlayer_Move就行，我们新建一个State，将其命名为Move，点进去，添加FirstPlayer_Move：   
![](https://i.imgur.com/vvV6JtS.png)     
回到Anim Graph，把线连上：    
![](https://i.imgur.com/DnHebZ1.png)   
回到VS，添加SlAiPlayerCharacter.cpp代码如下：   
![](https://i.imgur.com/Py3SkW8.png)       
   
2、**设置输入相关**      
（1）打开Project Settings，点开Input，Action Mappings和Axis Mappings的区别是Action Mappings是触发之后执行一次，而Axis Mappings是会传入值的。    
![](https://i.imgur.com/Ve2ZBS6.png)   
![](https://i.imgur.com/hjxlSGN.png)    
![](https://i.imgur.com/tu2QTof.png)     
![](https://i.imgur.com/Z8KHSEO.png)      
（2）现在我们回到VS，先添加SlAiPlayerCharacher.h代码如下：   
![](https://i.imgur.com/0c5VNUx.png)        
然后添加SlAiPlayerCharacher.cpp代码如下：   
![](https://i.imgur.com/1AspGaL.png)      
![](https://i.imgur.com/WAV2SjW.png)    
我们先看一下第三人称效果：   
![](https://i.imgur.com/NQZKfTU.png)     
接下来看一下运行的效果：    
![](https://i.imgur.com/P9NCI08.png)       
     
    
##10.2 玩家移动   
1、**继续写其他移动方法**    
（1）添加SlAiPlayerCharacter.h代码如下：   
![](https://i.imgur.com/CYalKzx.png)   
然后在SlAiPlayerCharacter.cpp进行初始化：       
![](https://i.imgur.com/AYl2lL5.png)   
再接着写其他的移动方法：   
![](https://i.imgur.com/CKeZvSo.png)   
![](https://i.imgur.com/0eT3pFw.png)     
我们回到构造函数那里，先添加头文件：    
![](https://i.imgur.com/ITG8ih3.png)    
然后设置初速度：   
![](https://i.imgur.com/BbNcpRy.png)   
再绑定动作：   
![](https://i.imgur.com/vpnsMqJ.png)      
之后设置想要显示的模型：   
![](https://i.imgur.com/Cbm01c7.png)   
但是我们在运行中发现并没有左右移动的动作，这是因为我们还没有绑定，添加SlAiPlayerAnim.h代码如下：     
![](https://i.imgur.com/D7bEZkw.png)    
再去添加cpp文件代码如下：   
![](https://i.imgur.com/SgqdsC7.png)    
我们再去SlAiPlayerAnim.h定义几个protected:   
![](https://i.imgur.com/IkBq6ez.png)   
再去cpp文件中实现他们：   
![](https://i.imgur.com/aJ755Lw.png)    
现在去SlAiThirdPlayerAnim.h重写更新属性方法:    
![](https://i.imgur.com/1ICGFm0.png)     
      
然后回到SlAiThirdPlayerAnim.cpp
![](https://i.imgur.com/BRtqlRn.png)     
然后再添加旋转部分的代码：     
![](https://i.imgur.com/NjT7PXY.png)     
现在我们就可以运行一下游戏看一下效果了：    
![](https://i.imgur.com/UZK9WVM.png)     
这样，人物移动就完成啦。