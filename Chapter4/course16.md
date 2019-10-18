#第十六节 准星，c++操控材质和射线检测伐木挖矿捡东西   
    
##16.1 准星与c++操控材质   
1、**准星**    
（1）打开ue4，我们先做一下材质,在Localization下新建一个文件夹命名威Material，在里面新建一个材质类并命名威PointerMat,打开，将它的Material Domain改为User Interface,然后将Blend Mode。接下来开始连接材质节点：    
![](https://i.imgur.com/kIS1hgQ.png)      
![](https://i.imgur.com/B9CIoNO.png)     
再添加一个0.6的浮点型，来获取遮罩，我们还需要找到Menulcon：       
![](https://i.imgur.com/5MnuREr.png)       
![](https://i.imgur.com/cLeOFAu.png)     
然后再将这个遮罩和圆相乘：   
![](https://i.imgur.com/ueo1BNZ.png)     
我们再获取这个进度条缺失的那一段：     
![](https://i.imgur.com/0xrALCY.png)     
最后将他们相加，设置为最终结果，再附加上遮罩：    
![](https://i.imgur.com/GwB18Co.png)   
现在我们改变它的进度条：   
![](https://i.imgur.com/n3u2zJr.png)    
之后我们需要在c++中调用它：    
![](https://i.imgur.com/P3I4LQZ.png)     
我们将它命名为PointerMatinst,打开勾选上Range,现在我们就要创建c++文件来调节进度条，我们New一个Slate Widget:     
![](https://i.imgur.com/ZHvqtxL.png)   
打开c++文件，在SSlAiGameHUDWidget.h下添加准星的引用：    
![](https://i.imgur.com/Jn13pVg.png)      
来到cpp文件实现它,先引入头文件SSlAiPointerWidget.h:     
![](https://i.imgur.com/VoMIdEH.png)        
我们再去SSlAiPointerWidget.h下添加如下代码：    
![](https://i.imgur.com/l6Dk5Fo.png)     
![](https://i.imgur.com/gyVo3Iy.png)     
再去cpp文件：     
![](https://i.imgur.com/yiwGrn8.png)    
![](https://i.imgur.com/ytw0dxB.png)
然后继续写SBox：    
![](https://i.imgur.com/A2caSso.png)     
    
2、**c++操控材质**
（1）接下来我们创建动态材质的指针：     
![](https://i.imgur.com/UglFa6S.png)    
再去cpp文件，先引入头文件：   
![](https://i.imgur.com/ygfQhFp.png)    
![](https://i.imgur.com/XueSkvU.png)     
再去.h文件添加如下代码更新准星：     
![](https://i.imgur.com/zX61ZR1.png)      
![](https://i.imgur.com/81F0nfJ.png)     
然后初始化一下这个变量：     
![](https://i.imgur.com/ZnBMmmS.png)     
再回到cpp文件添加如下代码实例化：   
![](https://i.imgur.com/JG9QFlM.png)   
我们重写一下Tick函数：     
![](https://i.imgur.com/KJwsqGH.png)     
我们去SlAiPlayerController.h下添加如下文件添加委托：    
![](https://i.imgur.com/4EiKEfI.png)     
然后声名：   
![](https://i.imgur.com/Ihwg1I4.png)   
然后去SlAiGameHUD.cpp绑定它，先引入一下头文件SSlAiPointerWidget.h：    
![](https://i.imgur.com/nEftisj.png)     
我们先看一下效果，去SlAiPlayerController.cpp下添加如下代码：    
![](https://i.imgur.com/zv9bIeL.png)   
现在就可以运行了，我们打开ue，去绑定这些笔刷：    
![](https://i.imgur.com/i76hGNY.png)    
   
##16.2 射线检测伐木挖矿捡东西    
1、**射线检测**    
（1）我们先添加一个射线类型：     
![](https://i.imgur.com/C4wyGF8.png)     
接下来我们设置碰撞：    
![](https://i.imgur.com/mtIkzJc.png)     
![](https://i.imgur.com/VcBE1Y9.png)     
这样碰撞我们就添加完了，回到vs，删除以前的临时代码，删SlAiPlayerController.cpp下的代码：     
![](https://i.imgur.com/g5rk9PO.png)   
然后就可以去SlAiPlayerController.h添加射线检测了：    
![](https://i.imgur.com/g12a4e4.png)    
然后去cpp文件实现这三个方法，先添加头文件Components/LinBatchComponent.h,Camera/CameraComponent.h：    
![](https://i.imgur.com/L4HnJlL.png)     
![](https://i.imgur.com/07ZVq3P.png)    
![](https://i.imgur.com/MiOq5lA.png)    
![](https://i.imgur.com/2SO6Uo6.png)   
运行游戏可以看到射线已经画出来了：    
![](https://i.imgur.com/4mZ7vpU.png)    
（2）接下来我们做检测物品，先去SlAiPlayerController.h下添加指针：    
![](https://i.imgur.com/AoKrLPr.png)    
再在SlAiPickObject.h的public添加获取物品名字的方法，先引入头文件SlAiTypes.h：    
![](https://i.imgur.com/XkPFzbQ.png)   
然后去cpp文件实现它：    
![](https://i.imgur.com/EgPZSUH.png)    
之后我们去到SlAiPlayerState.h定义一个变量：    
![](https://i.imgur.com/sbNZVcU.png)    
去cpp文件中修改代码如下来实现获取名字：    
![](https://i.imgur.com/DUoUoLv.png)     
去到SlAiPlayerController.cpp下添加如下代码，同时添加ASlAiPickupObject的头文件：     
![](https://i.imgur.com/UeATW7I.png)     
然后再添加如果检测为空的情况：     
![](https://i.imgur.com/557z5nL.png)     
（1）现在我们要动态显示准星，那么我们就要去SlAiPlayerController.h下再添加一个函数：    
![](https://i.imgur.com/tCO5ZEX.png)    
然后去cpp文件中实现它，同时删除以前写的临时代码：    
![](https://i.imgur.com/63SMLds.png)     
![](https://i.imgur.com/WUHSEzA.png)    
接下来我们要设定攻击范围，我们去到SlAiPlayerState.h添加如下代码：     
![](https://i.imgur.com/y3zqn8o.png)    
然后再去cpp文件实现它：    
![](https://i.imgur.com/UG0DcR0.png)    
但是我们还没有定义血量，我们去到SlAiResourceObject.h:    
![](https://i.imgur.com/M0hgFP7.png)    
然后去cpp文件：    
![](https://i.imgur.com/cqVESbG.png)    
回到.h文件添加三个方法：     
![](https://i.imgur.com/wOoNiws.png)
现在去实现他们，先引入头文件，Engine/GameEngine.h，然后添加如下代码：     
![](https://i.imgur.com/xAj9jFi.png)    
![](https://i.imgur.com/J9Ir6nw.png)    
我们还需要获取伤害值，在SlAiPlayerState.h下添加如下代码：    
![](https://i.imgur.com/lSgKeKR.png)     
然后再去cpp文件实现它：    
![](https://i.imgur.com/d67lM3u.png)     
现在我们就可以回到SlAiPlayerController.cpp继续写了：      
![](https://i.imgur.com/GtSSvyl.png)       
最后修改SlAiResourceObject.cpp代码如下：     
![](https://i.imgur.com/1QCbycj.png)     
我们可以先运行看一下效果：    
![](https://i.imgur.com/BNYnA8y.png)      
现在我们再把射线的代码注释掉，来到SlAiPlayerController.cpp:    
![](https://i.imgur.com/2SAptKs.png)    
2、**可拾取物品**    
（1）我们先去SlAiPickupObject.h添加一个方法：    
![](https://i.imgur.com/Ekvo7Yj.png)   
然后去实现它，先引入头文件，Engine/GameEngine.h：    
![](https://i.imgur.com/YUSstE1.png)    
添加SlAiPlayerController.cpp代码如下：    
![](https://i.imgur.com/CZUBcbM.png)      
然后就可以运行了。