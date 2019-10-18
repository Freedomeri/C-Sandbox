#第一节 资源导入及虚幻纯C++开发环境搭建
  

- 了解不同引擎版本之间资源的同步；  
- 了解自定义C++工程的创建和基本框架；  
- 了解虚幻引擎C++类的编程规则。
  
##1.1新建工程  
1、新建c++工程，选择没有初学者内容。  
注：在用虚幻4编辑器调试运行独立游戏时，选择下图方式：  
<center>![](https://i.imgur.com/ecJjfRP.png)</center>
  

##1.2资源获取  
1、本教程提供4.19版本的资源包。若最新版本引擎高于此，则用如下方法将资源合并至最新版引擎中。  
<center>![](https://i.imgur.com/tj1mOCe.png)</center>  

##1.3安装VAssistX插件    

一个很方便的VS插件，自行下载安装。

##1.4类命名前缀  
&emsp;&emsp;虚幻引擎为用户提供在构建过程中生成代码的工具。这些工具拥有一些类命名规则。如命名与规则不符，将触发警告或错误。下方的类前缀列表说明了命名的规则。  
&emsp;&emsp;派生自 Actor 的类前缀为 **A**，如 AController。  
&emsp;&emsp;派生自 对象 的类前缀为 **U**，如 UComponent。  
&emsp;&emsp;枚举 的前缀为 **E**，如 EFortificationType。  
&emsp;&emsp;接口 类的前缀通常为 **I**，如 IAbilitySystemInterface。  
&emsp;&emsp;模板 类的前缀为 **T**，如 TArray。  
&emsp;&emsp;派生自 SWidget（Slate UI）的类前缀为 **S**，如 SButton。  
&emsp;&emsp;其余类的前缀均为 字母 **F** ，如 FVector。  

&emsp;&emsp;更详细的虚幻编程规则可以访问[https://docs.unrealengine.com/en-us/Programming/Development/CodingStandard#namingconventions](https://docs.unrealengine.com/en-us/Programming/Development/CodingStandard#namingconventions)  
&emsp;&emsp;中文文档：[http://api.unrealengine.com/CHN/Programming/Introduction/index.html](http://api.unrealengine.com/CHN/Programming/Introduction/index.html)