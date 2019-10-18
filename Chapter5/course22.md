#第二十二节  合成台功能的实现和添加物品到背包
 
##22.1 合成台功能实现
1、**数据结构**<br/>
&ensp;&ensp;(1)打开SLAITypes.h头文件向其中添加如下代码实现合成表结构体：![](https://i.imgur.com/hVmrU66.png)
&ensp;&ensp;(2)然后在SLAIJsonHandle.h对合成表文件进行读取，我们先定义一个合成表文件名：<br/>![](https://i.imgur.com/MT0KruO.png)<br/>
&ensp;&ensp;(3)同时新定义一个合成表的方法：![](https://i.imgur.com/nkZnoJC.png)<br/>
&ensp;&ensp;(4)在SLAIJsonHandle.cpp中对新建的合成表的方法进行实现并在构造方法中指定合成表的文件名：![](https://i.imgur.com/MJ9Cw7Z.png)<br/><br/>![](https://i.imgur.com/MbPVkMQ.png)
&ensp;&ensp;(5)在SLAIJsonHandle.h文件中定义一个合成表的图的变量和方法![](https://i.imgur.com/m3fxYXP.png):![](https://i.imgur.com/JIA1QB1.png)
&ensp;&ensp;(6)在cpp文件进行实现,在InitializeGameData中进行调用![](https://i.imgur.com/MBcM9FN.png)：![](https://i.imgur.com/14o28sQ.png)
&ensp;&ensp;(7)在SLAITypes.h中定义检测符合表中物品ID和数量的方法：![](https://i.imgur.com/A7ORE2N.png)
&ensp;&ensp;(8)在ALAIPackageManager.h中添加检测是否可以叠加的方法：![](https://i.imgur.com/X499X4c.png)<br/>
&ensp;&ensp;(9)在cpp文件中进行实现：
![](https://i.imgur.com/AkQfVhc.png)
&ensp;&ensp;(10)实现合成表的输入:![](https://i.imgur.com/wU3HIJW.png)
&ensp;&ensp;(11)在SLAITypes.h中实现物品输入的物品ID和输出ID序列以及胜场数量查询出是否匹配这个合成表并且消耗数组![](https://i.imgur.com/1uF4rUS.png)
&ensp;&ensp;(12)实现合成表的输出:![](https://i.imgur.com/FFTBGgW.png)
&ensp;&ensp;(13)运行观察效果。
##22.2 添加物品到背包
1、**物品添加到背包**<br/>
&ensp;&ensp;(1)在SSLAIContainerBaseWidget.h文件中添加检测背包是否为空的方法并在cpp进行实现:<br/>
![](https://i.imgur.com/ylYCKb9.png)
![](https://i.imgur.com/du0rd1r.png)
&ensp;&ensp;(2)在SLAIPackageManager.h文件中添加提供给外部访问是否有插入空间、添加物品、添加元素的方法并实现:
![](https://i.imgur.com/IEXfSeT.png)

![](https://i.imgur.com/URJq0pg.png)
&ensp;&ensp;(3)在SLAIPlayerCharacter.h中添加检测背包是否有空间和添加物品到背包的方法并实现：![](https://i.imgur.com/gdXDJjd.png)<br/><br/>![](https://i.imgur.com/bspf0VX.png)
&ensp;&ensp;(4)将Tick中的代码改变到如图所示:![](https://i.imgur.com/ASrWobO.png)
&ensp;&ensp;(5)将下列代码也进行相应的修改：
![](https://i.imgur.com/h8cuRla.png)
&ensp;&ensp;(6)将SLAIPlayerController.cpp中的StateMachine代码改为以下代码：![](https://i.imgur.com/tbTUGAB.png)
&ensp;&ensp;(7)将SLAIPlayerCharacter.h中添加吃完东西调用的方法：![](https://i.imgur.com/g1SsTXp.png)
&ensp;&ensp;(8)在SLAIPackgeManager.h中添加是否成功吃掉食物的方法：![](https://i.imgur.com/gYf28Fg.png)<br/>
&ensp;&ensp;(9)实现SLAIPlayerManager.cpp中的EatupEvent方法：![](https://i.imgur.com/epd42Tz.png)
&ensp;&ensp;(10)在Beginplay中添加如下代码:![](https://i.imgur.com/gzW3mMG.png)
&ensp;&ensp;(11)在SLAIPlayerState.h中添加提升饥饿值的方法并进行实现：![](https://i.imgur.com/EW4ShwK.png) ![](https://i.imgur.com/WhQ4iPT.png)
&ensp;&ensp;(12)在动画蓝图中添加![](https://i.imgur.com/Tb2Zj2U.png)