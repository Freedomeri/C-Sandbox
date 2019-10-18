#第二十七节 敌人受伤AI、逃跑与防御AI  

##27.1 敌人受伤AI  

1、首先打开虚幻引擎，再基于TaskBase类新建C++任务类：  

![](https://i.imgur.com/Xt80IcY.png)  

2、打开VS工程，先做些准备工作。  

首先到EnemyCharacter.h中public下声明接受伤害的方法：  

![](https://i.imgur.com/N3leyTW.png)  

然后到.cpp中实现一部分：  

![](https://i.imgur.com/N9E636i.png)  

由于敌人的死亡还没有写，所以if条件成立的部分先空下。而血值不为0时，我们需要告诉EnemyController进行相关操作。  

3、来到EnemyController.h，定义一个private变量：  

![](https://i.imgur.com/mYvGurc.png)  

然后定义一个public函数：  

![](https://i.imgur.com/xS7u76s.png)  

然后到.cpp中实现一下：  

![](https://i.imgur.com/baKXQdc.png)  

写到这里我们发现，不能让敌人一受伤就播放动画，否则玩家攻击频率很高的话会出现不合理的情况。  

回到.h文件，再定义两个private变量：  

![](https://i.imgur.com/egapW0o.png)  

并在BeginPlay函数中将定义的变量初始化：  

![](https://i.imgur.com/VPN8KiB.png)  

然后在更新状态的函数UpdateStatePama函数中添加代码：  

![](https://i.imgur.com/EDpsmFN.png)  

最后现在继续完善UpdateDamageRatio函数：  

![](https://i.imgur.com/hYdPIWJ.png)  

这些逻辑就使得敌人在6秒之内只会受伤一次。  

4、去到SlAiHandObject.cpp中，重新写交互函数OnOverlayBegin：  

![](https://i.imgur.com/toTt7wH.png)  

5、交互之后要播放敌人受伤的动画。  
到EnemyAnim.h中，先定义private动画指针：  

![](https://i.imgur.com/NmYtajf.png)  

再定义播放动画的public函数：  

![](https://i.imgur.com/8j55uM1.png)  

去到.cpp文件中，先在构造函数中获取Montage动画资源：  

![](https://i.imgur.com/2v7FBr4.png)  

然后实现播放受伤动画的函数：  

![](https://i.imgur.com/eiBuuDb.png)  

6、由于任务节点是通过Character播放相应动画，所以去到EnemyCharacter.h中，定义public函数：  

![](https://i.imgur.com/yi96V3P.png)  

然后到.cpp中实现一下：  

![](https://i.imgur.com/Y4aw61M.png)  

7、接下来正式写受伤任务逻辑。  

去到EnemyTaskHurt.h中，定义所需函数和变量：  

![](https://i.imgur.com/raJu7ZO.png)  

![](https://i.imgur.com/MMIbGNB.png)  

来到.cpp文件中，同样是先引入所需头文件：  

![](https://i.imgur.com/pimQIs6.png)  

然后实现的方法同之前的任务相似：  

![](https://i.imgur.com/yeEs376.png)  

![](https://i.imgur.com/rTKtCcP.png)  

AbortTask方法也同之前的相似：  

![](https://i.imgur.com/yflu4LP.png)  

8、在OnAnimationTimerDone中要实现受伤动画到其他动画的跳转。实现OnAnimationTimerDone函数前，先去到EnemyController.h中定义public函数：  

![](https://i.imgur.com/AamUgWW.png)  

我们将在这个函数中完成判断敌人在不同状态下受伤时播放的动画。  

然后在EnemyController.cpp中实现：  

![](https://i.imgur.com/bqWRPD5.png)  

![](https://i.imgur.com/63xJ6o1.png)  

![](https://i.imgur.com/VLpIWSJ.png)  

上面是血量低于20%时触发的事件。下面是高于20%时：  

![](https://i.imgur.com/7BUfaKq.png)  

![](https://i.imgur.com/Bm6XVPY.png)  

9、去到EnemyCharacter.cpp中，完成之前的AcceptDamage函数的血值大于0部分：  

![](https://i.imgur.com/diElnf8.png)  

10、最后回到EnemyTaskHurt.cpp，在OnAnimationTimerDone函数中调用Controller中的受伤方法：  

![](https://i.imgur.com/eYIHSOk.png)  

11、代码已经写完了，打开虚幻编辑器，打开EnemyBehaviorTree行为树，在Hurt节点下添加两个任务：  

![](https://i.imgur.com/1J9fM33.png)  

Save一下，回到场景中，我们想要测试敌人受伤动画的话，还要有一个武器。所以我们直接在场景中放一堆木头和石头，自己合成。在Pickup文件夹中拖进场景即可：  

![](https://i.imgur.com/AtYxoSS.png)  

![](https://i.imgur.com/z6ag6v8.png)  

12、点击Play进入游戏，我们捡一些木头和石头，并在合成表中按下图所示合成一把剑，拿到手上。  

![](https://i.imgur.com/Rc7VYvA.png)  

运行游戏，可以发现当我们攻击敌人的时候，血条会慢慢减少，并且每6秒最多有一次的受伤动画。  

##27.2 逃跑与防御AI  

1、首先去到EnemyAnim.h中，定义一个public变量，在动画蓝图中会使用到：  

![](https://i.imgur.com/buoFskL.png)  

然后到.cpp的构造函数中初始化为false。  

2、直接点击本地调试器，运行虚幻编辑器。打开Blueprint/Enemy文件夹下的动作蓝图，双击进入LocalMotion的界面，在Move后面Add 一个State，命名为DefenceStart。  

![](https://i.imgur.com/qyYfIh7.png)  

点击进入DefenceStart，拖动右侧的DefenceStart动画到中间，并连接到Final：  

![](https://i.imgur.com/xG6jbzL.png)  

回到LocalMotion，双击Move到DefenceStart之间的连线，把条件设置为我们一开始在C++里定义的bool值。  

![](https://i.imgur.com/a8z8BdD.png)  

3、用同样的方式，在DefenceStart后面添加一个State，命名为DefenceLoop。  

![](https://i.imgur.com/eSdwJEs.png)  

![](https://i.imgur.com/Ux4Q9vz.png)  

![](https://i.imgur.com/x35gA3v.png)  

我们让DefenceStart动画剩余时间大于10%的时候就开始进入Loop状态。  

4、再用同样的方式，在DefenceLoop后面添加一个状态：  

![](https://i.imgur.com/Yn6aL0y.png)  

![](https://i.imgur.com/lfR5lIo.png)  

![](https://i.imgur.com/jPZLLyQ.png)  

最后把DefenceEnd连到Move上，并把条件设置为End动画只剩10%时：  

![](https://i.imgur.com/FiTEsgU.png)  

最后点击Compile并点击Save。完成编辑。  

5、接下来创建与逃跑和防御有关的C++文件：  

还是以TaskBase为基类，创建第一个：  

![](https://i.imgur.com/SoUyBbU.png)  

第二个：  

![](https://i.imgur.com/OC6vt0y.png)  

最后一个锁定逃跑目的地的任务类：  

![](https://i.imgur.com/fxvJKXi.png)  

到这里，所有的任务类已经创建完了。  

6、回到VS工程，来到EnemyCharacter.h，定义一个public防御有关函数：  

![](https://i.imgur.com/eXDaiYv.png)  

在.cpp中实现：  

![](https://i.imgur.com/4XyISpS.png)  

7、开始写防御任务类。  
来到TaskDefence.h中，定义相关函数和变量：  

![](https://i.imgur.com/u3GuQl5.png)  

![](https://i.imgur.com/no1FxGO.png)   

然后到.cpp中实现：  

![](https://i.imgur.com/u1eHRgE.png)  

![](https://i.imgur.com/6nSefJ1.png)  

![](https://i.imgur.com/jxqKwN0.png)  

8、在写动画完成的函数时，还是要先去到EnemyController.h中，定义public函数：  

![](https://i.imgur.com/ShmoV4J.png)  

在实现之前，我们还需要知道在防御结束时，玩家是否还在攻击，所以先去到PlayerCharacter.h中，定义public变量：  

![](https://i.imgur.com/CoBoWlu.png)  

并在构造函数中初始化：  

![](https://i.imgur.com/ESebWZF.png)  

然后在下面找到ChangeHandObjectDetect函数，添加代码：  

![](https://i.imgur.com/yJ66LqO.png)  

这样就使得在玩家武器与敌人进行交互检测的时候，把攻击状态改为true。  

9、接下来回到EnemyCharacter.h中，定义停止防御的方法：  

![](https://i.imgur.com/qeTIrVT.png)  

并在.cpp中实现：  

![](https://i.imgur.com/ReAXwjB.png)  

10、回到EnemyController.cpp，实现FinishStateDefence函数：  

![](https://i.imgur.com/2L2VnIM.png)   

![](https://i.imgur.com/X8jfnYV.png)  

11、最后回到EnemyTaskDefence.cpp，实现最后一个函数：  

![](https://i.imgur.com/RUdv0gx.png)  

12、然后我们在防御状态的时候不能让敌人掉血，所以来到EnemyCharacter.cpp中，在AcceptDamage函数添加代码：  

![](https://i.imgur.com/ztIbpCh.png)  

防御的就先写到这里。有兴趣的可以先打开引擎调试一下看效果，这里直接继续写逃跑的任务。   


13、来到EnemyTaskEscapeSwitch.h，定义所需函数和变量：  

![](https://i.imgur.com/nCgZK8s.png)  

并到.cpp中实现：  

![](https://i.imgur.com/gV81Fzo.png)  

![](https://i.imgur.com/E3nR2Ff.png)  

14、接下来是锁定逃跑目的地的函数。  
来到EnemyTaskLockESC.h中，定义相关函数变量：  

![](https://i.imgur.com/t3vwfHM.png)  

然后到.cpp中，添加完头文件，开始实现执行任务的函数：  

![](https://i.imgur.com/asu5oKW.png)  

![](https://i.imgur.com/9p3piKg.png)  

![](https://i.imgur.com/barLvBz.png)  

当玩家到敌人的向量与敌人到目的地的向量夹角大于90度时，说明敌人不是背离玩家逃跑的，所以要重新获取随机点。  

15、来到EnemyCharacter.cpp，在BeginPlay函数中把HP改回满血200：  

![](https://i.imgur.com/JYtUqle.png)  

16、接下来是一些之前任务的优化。来到TaskAttackNormal.h，把PlayerPawn这个变量删掉。并在.cpp中删掉相关语句：  

![](https://i.imgur.com/3cHO6up.png)  

![](https://i.imgur.com/UrHjTmW.png)

检查所有的任务类都添加好头文件后，打开虚幻编辑器。  

16、打开EnemyBehaviorTree行为树，来到普通攻击的那个节点下，把中间的那个Rotate节点删掉。然后右击Sequence节点，Add Service->Default Focus，并把focus的BlackboardKey设为PlayerPawn：  

![](https://i.imgur.com/qtVEhJP.png)  

使用这个Service，就可以很方便地实现敌人在普通攻击时一直面朝玩家的效果。  

17、然后连逃跑的节点：  

![](https://i.imgur.com/DqCrK58.png)  

18、还有Defence防御的节点：  

![](https://i.imgur.com/MWtlceO.png)  

  

其中要把TaskDefence节点的Blackboard设置一下：  

![](https://i.imgur.com/Cms9C1K.png)  

Loop和Focus的节点属性请根据图片自行设置。  

19、保存，运行游戏。合成一把剑后，去跟敌人打。会看到打几下敌人就会摆出防御姿态，并且打到20%血之后敌人会逃跑。  




  



