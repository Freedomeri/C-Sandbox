#第六节 模板单例与Json存档的读写  

##6.1 基于模板template的单例模式  

1、首先我们需要在Build.cs中添加Json模块：  

![](https://i.imgur.com/jtDMhTI.png)  

然后打开虚幻编辑器，创建两个无继承的C++类。Singleton是基于模板的单例模式类，一般在整个程序运行中只有唯一一个实例对象。此游戏中我们用来做跟存档数据相关的事。  

![](https://i.imgur.com/KclGhU9.png)  

第二个类是用来处理Json文件数据的：  

![](https://i.imgur.com/nJuG45q.png)  

2、把单例模板类所需成员声明：  

![](https://i.imgur.com/QouDPjC.png)  

模板类的实现尽量放在.h文件中，而不是.cpp文件中：  

![](https://i.imgur.com/RQpWh1E.png)  

![](https://i.imgur.com/Y24S1CJ.png)  


  模板是C++实现代码重用机制的一种工具。当一个类中某些数据成员的类型不确定，或成员函数的参数及返回值不确定，则可以使用**template < class T >**格式将其定义。其中class可以用typename替换；"T"可以用任意名称替换。  

最常见的例子是：求两数最大值，可以有如下几种形式Max(int , int) , Max(float , float) , Max(double , double)  
这种不同的重载形式一是会比较麻烦，二是如果使用其他的数据类型作为参数（如char），则无法完成。  

3、在SlAiDataHandle.h中声明两个函数，用来把语言的枚举值跟字符串相互转换（只可用于虚幻的UENUM宏定义的枚举。普通枚举可通过switch实现）：  

![](https://i.imgur.com/u90VU6G.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/AdkxLRa.png)  

![](https://i.imgur.com/YBbP8QQ.png)  
FindObject函数从ANY_PACKAGE中寻找名为Name（在后面章节会传入语言的枚举类名）的UEnum反射（这也是为什么普通枚举类不能使用这个方法），如果找不到，第二个函数会返回一个默认的枚举值0，即英文的枚举值。  

![](https://i.imgur.com/blkOx4i.png)  
（图为两种不同枚举）  


4、读取文件  
在SlAiJsonHandle.h中定义两个私有变量：  

![](https://i.imgur.com/E0FrnUV.png)  

在.cpp文件中初始化两个变量，初始化为Json文件的路径  

![](https://i.imgur.com/TJnbU7E.png)  

回到.h文件中，定义读取Json文件的私有函数：  

![](https://i.imgur.com/9l5cOkp.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/In9tKMs.png)  

![](https://i.imgur.com/RHPJWVd.png)  




##6.2 Json文件的读  

1、Json是一种轻量级数据交换格式。类似于Xml格式的性质。本游戏中的存档均使用Json格式。详细内容请自行搜索。  

2、存档数据解析  
在SlAiJsonHandle.h中声明public数据解析函数：  

![](https://i.imgur.com/GNOI0Nz.png)   

其中的参数在Json文件中都有对应：  
![](https://i.imgur.com/24v0Iy4.png)  

然后在.cpp文件中实现此函数：  

![](https://i.imgur.com/sU3BmWF.png)  

![](https://i.imgur.com/oNlfE8T.png)  

![](https://i.imgur.com/aEXQ6eM.png)  

![](https://i.imgur.com/l8draIq.png)  
  
在通过路径寻找文件时，用FPaths类的GameContentDir()函数，进到Content目录下。再加上相对路径"Res/ConfigData/"，最后加上文件名。整个FString字符串组成了Json数据文件的路径。  
内置的FFileHelper类的LoadFileToString()函数给我们提供了加载文件中字符串的方法。  

RecordDataJsonRead()函数的
思路是，先将Json文件的数据读取到JsonValue字符串变量中，利用TJsonReaderFactory< TCHAR >::Create()函数创建解析器，最后通过FJsonSerializer::Deserialize()方法将Json数据解析到JsonParsed数组中。  

4、在DataHandle中初始化读取存档数据  

在SlAiDataHandle.h中声明私有的初始化函数：  

![](https://i.imgur.com/O85q6nx.png)  

再声明一个保存存档数据的公有数组：  
![](https://i.imgur.com/geHEjmv.png)  

然后在.cpp文件中实现：  

![](https://i.imgur.com/vnYar0Q.png)![](https://i.imgur.com/5mzOKkF.png)  

![](https://i.imgur.com/rO1vVwv.png)  


  
TArray< FString >::Iterator It()声明了一个迭代器It。  
迭代器是一种检查容器元素并遍历容器元素的数据类型。这里容器是TArray，字符串类型的数组。对RecordDataList里面的每一个元素进行检查并执行Debug函数。  
![](https://i.imgur.com/c5hNpHf.png)  

  


在初始化语言的过程中，由于我们获取到的是FString类型的语言数据  
![](https://i.imgur.com/NyxPCRM.png)  

而设置语言需要我们定义的枚举类型  
![](https://i.imgur.com/NkcgyVF.png)  
这时就用到了我们之前写的转换函数，将字符串转换为枚举类型的数据。
![](https://i.imgur.com/kKpyTYF.png)  

##6.3 Json文件的写  

1、首先在SlAiJsonHandle.h中声明保存字符串到文件的私有函数：  

![](https://i.imgur.com/3MTvwFW.png)  

然后到.cpp文件中实现：  

![](https://i.imgur.com/RroIvgL.png)  

![](https://i.imgur.com/ddUyQx4.png)  

2、再声明一个将Json对象转换为字符串的private函数：  

![](https://i.imgur.com/QHTQskp.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/cUvIlMv.png)  

3、还要声明一个public的函数，用来给DataHandle调用，将数据传进来，转换成JsonObject：  

![](https://i.imgur.com/fkqVwfJ.png)  

在.cpp文件中实现：  

![](https://i.imgur.com/X7ooPuD.png)  

Culture语言数据的转换，我们先定义一个语言相关的JsonObject，然后设置它里面"Culture"关键字下的内容为传入的Culture，最后定义一个JsonValueObject将数据对象保存起来。  
同理，音乐音效等数据也是这样转换：  

![](https://i.imgur.com/zGiQMrQ.png)  

存档信息由于有多条，需要再定义数组实现：  
![](https://i.imgur.com/W0cS5ql.png)  

然后将所有数据对象保存到一个对象数组中：
![](https://i.imgur.com/McYd6xm.png)  

最后去掉多余的字符（已计算好字符的位置），并写入json文件：  

![](https://i.imgur.com/V9IZUaT.png)  


4、在DataHandle中调用函数  

有两个地方需要调用刚才写的函数，第一个是初始化时，下面代码加在ChangeLocalizationCulture函数里面：  

![](https://i.imgur.com/gThXrmu.png)  

第二个地方是重设菜单选项时（将上面代码复制到此处即可）：  

![](https://i.imgur.com/4szRFCk.png)  


5、打开虚幻编辑器，运行游戏。对比看两段Debug，上面部分是修改之后的，下面是修改之前的。可以看到修改之前有多余字符串，如果放在Json文件里，虚幻引擎不会识别正确。  
当我们修改音量或语言时，上方Debug也会实时显示修改之后的数据。而这些数据都已经写入到Json文件中。  

![](https://i.imgur.com/7ZvxBV5.jpg)  

没有问题之后，为了不影响美观，可以回到VS工程中，将这两段Debug注释掉了。