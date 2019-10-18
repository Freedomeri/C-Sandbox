#第二十五节 AI的实现和动画的播放

#22.1 敌人追逐AI
1、**数据结构代码**
&ensp;&ensp;(1)在ALAIEnemyCharacter.cpp将代码改为如图所示：![](https://i.imgur.com/4re68eO.png)
&ensp;&ensp;(2)在SLAIType.h中添加新的数据结构来判定敌人的攻击状态：![](https://i.imgur.com/1ufSlG7.png)
&ensp;&ensp;(3)新创建两个C++)类LAIEnemyTaskChaseSwitch和SLAIEnemyTaskLocaSP:![](https://i.imgur.com/TQwfw2U.png)<br/>![](https://i.imgur.com/4Py05IC.png)
&ensp;&ensp;(4)在SLAIPlayerState.h文件中新定义方法并在cpp文件中进行实现：![](https://i.imgur.com/MGwHLmi.png)<br/><br/>![](https://i.imgur.com/xrNJDCc.png)
&ensp;&ensp;(5)在SLAIPlayerCharacter.h中新定义方法并在cpp中实现： ![](https://i.imgur.com/cAclCYm.png)
&ensp;&ensp;(6)在SLAIEnemyCharacter.h中新定义方法并在cpp中实现:![](https://i.imgur.com/P2tfnuO.png)<br/><br/>![](https://i.imgur.com/mf3YwRV.png)<br/><br/>![](https://i.imgur.com/7K0SDpo.png)
&ensp;&ensp;(7)在SLAIEnemyController.h中新定义变量并在BeginPlay方法中进行初始化：<br/>
![](https://i.imgur.com/CA3ScIZ.png)<br/><br/>![](https://i.imgur.com/7Vdo54O.png)
&ensp;&ensp;(8)在SLAIEnemyChaseSwitch.h中新定义方法和变量:![](https://i.imgur.com/xb10fb7.png)
&ensp;&ensp;(9)在SLAIEnemyController.h中新定义方法并进行实现：![](https://i.imgur.com/PPfbrW3.png)<br/><br/>![](https://i.imgur.com/7k4xtQD.png)
&ensp;&ensp;(10)在SLAIEnemyTaskLocaSP.h中新定义变量和方法:![](https://i.imgur.com/SpMauv3.png)<br/><br/>![](https://i.imgur.com/2MCNHHh.png)
2、**UE4**
&ensp;&ensp;(1)在行为树中添加如下分支：![](https://i.imgur.com/oiN2VB4.png)
&ensp;&ensp;(2)改变OnSeePlayer方法中的代码如下并在BeginPlay中进行初始化：![](https://i.imgur.com/nisEula.png)<br/><br/>![](https://i.imgur.com/eeI2zHL.png)
#22.2 C++动画RootMotion
1、**播放动画**<br/>
&ensp;&ensp;(1)在SLAIEnemyAnim.h中添加动画指针并在cpp文件中进行获取：<br/>![](https://i.imgur.com/hsxWiIi.png)<br/><br/>![](https://i.imgur.com/FScHp4R.png)
&ensp;&ensp;(2)在SLAIEnemyAnim.h中新定义两个变量并进行初始化:![](https://i.imgur.com/d7c7W7q.png)<br/><br/>![](https://i.imgur.com/O5hNmii.png)
&ensp;&ensp;(3)对NativeUpdateAnimation方法中的代码进行修改如下：![](https://i.imgur.com/VJxXBrx.png)
&ensp;&ensp;(4)继续新定义变量并在cpp文件中进行初始化：![](https://i.imgur.com/pULLCcv.png)<br/><br/>![](https://i.imgur.com/p0r07lj.png)
&ensp;&ensp;(5)改变NativeUpdateAnimation方法中添加代码如下：![](https://i.imgur.com/GuzrDd3.png)
&ensp;&ensp;(6)在头文件中新定义方法并进行实现：
![](https://i.imgur.com/YGuesqk.png)<br/><br/>![](https://i.imgur.com/IT6DpOW.png)![](https://i.imgur.com/JnJR8Au.png)
&ensp;&ensp;(7)在NativeUpdateAnimation方法添加如下代码：![](https://i.imgur.com/oHpqw8g.png)
&ensp;&ensp;(8)在头文件中添加PlayerAttackAction方法并进行实现：![](https://i.imgur.com/PfxSFWm.png)<br/><br/>![](https://i.imgur.com/QACsA2C.png)
&ensp;&ensp;(9)创建如下类：![](https://i.imgur.com/Y6Itwjd.png)![](https://i.imgur.com/6cVBbs6.png)![](https://i.imgur.com/KDRDmIU.png)![](https://i.imgur.com/oRKShyW.png)![](https://i.imgur.com/qWHLzvq.png)![](https://i.imgur.com/Y6pANAC.png)