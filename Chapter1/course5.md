#第五节Debug与游戏设置控件
##5.1 自定义Debug功能
1.**添加Debug功能**  
首先创建一个SlAiInternation空白类，放在Common文件夹下。在其中实现Debug功能。  
![](https://i.imgur.com/MaCwMhM.png)


删掉SlAiInternation的.cpp文件，只需要.h文件。 

在Helper头文件内引入头文件。  
```
#include "Engine/GameEngine.h"
#include "Engine/world.h"
```
并建立一个全局的方法（在屏幕上打印一段黄色的文字，持续`Duration`秒）
```
namespace SlAiHelper {

	FORCEINLINE void Debug(FString Message, float Duration = 3.f) {
		if (GEngine) {
			GEngine->AddOnScreenDebugMessage(-1, Duration, FColor::Yellow, Message);
		}
	}
```
![](https://i.imgur.com/44pX8Ss.png)

FORCEINLINE是UE4的内联函数。   
*内联函数适用情况：  
1.一个函数被重复调用  
2.函数只有几行，且不包含for，while，switch语句。   
内联函数应该放在头文件中定义，这一点不同于其他函数。*  

2.**调用Debug功能**
在`SSlAiMenuWidget.cpp`中插入头文件
	#include "SlAiHelper.h"
在`MenuItemOnClicked`函数中插入
`SlAiHelper::Debug(FString("hhhh"), 5.f);`
![](https://i.imgur.com/y4kzViV.png)

最终效果：
![](https://i.imgur.com/Q3CUEFw.jpg)


##5.2游戏设置控件## 
菜单预览图
![](https://i.imgur.com/4M7X97C.png)
1.**创建中英文切换的控件**
创建新的Slate Widget类
![](https://i.imgur.com/YxJG58y.png)

在`SSlAiMenuWidget.cpp`中注释掉`MenuItemOnClicked`函数中Debug功能，防止不必要的错误
引入头文件`#include "SSlAiGameOptionWidget.h"`
并删除原本在`ContentBox`下创建的`Start game`按钮改为创建`SSlAiGameOptionWidget`内的Item
替换代码为：
`SNew（SSlAiGameOptionWidget）`
![](https://i.imgur.com/IsKzQBl.png)

(1)先在`SSlAiGameOptionWidget`中获取样式
在cpp文件中插入头文件
```
#include "SlAiStyle.h"
#include "SlAiMenuWidgetStyle.h"
#include "SBox.h"
#include "SImage.h"
#include "SBorder.h"
#include "SOverlay.h"
#include "STextBlock.h"
#include "SBoxPanel.h"
#include "SCheckBox.h"
#include "SlAiDataHandle.h"
```
在`.h`文件中新建指针变量
`const struct FSlAiMenuStyle *MenuStyle;`
在构造函数中插入
`MenuStyle = &SlAiStyle::Get().GetWidgetStyle<FSlAiMenuStyle>("BPSlAiMenuStyle");`
并引入头文件
`#include "SlAiTypes.h"`

(2)在`SlAiMenuWidgetStyle.h`中定义笔刷和字体颜色
```
//黑色颜色
UPROPERTY(EditAnywhere, Category = Common)
	FLinearColor FontColor_White;
//白色颜色
UPROPERTY(EditAnywhere, Category = Common)
	FLinearColor FontColor_Black;
//GameOption的背景
UPROPERTY(EditAnywhere, Category = GameOption)
	FSlateBrush GameOptionBGBrush;
//CheckedBox的Brush被选中
UPROPERTY(EditAnywhere, Category = GameOption)
	FSlateBrush CheckedBoxBrush;
//CheckedBox的Brush不被选中
UPROPERTY(EditAnywhere, Category = GameOption)
	FSlateBrush UnCheckedBoxBrush;
//音量条背景
UPROPERTY(EditAnywhere, Category = "GameOption")
	FSlateBrush SliderBGBrush;
//音量按钮
UPROPERTY(EditAnywhere, Category = "GameOption")
	FSliderStyle SliderStyle;
```
![](https://i.imgur.com/BsMooPr.png)

(3)在`ChildSlot`下新建一个SBox限制整体大小
再新建`SOverlay`添加背景图片和一个垂直布局`SVerticalBox`
垂直布局分为三行`.FillHeight`传入参数为`1.f`即100%填充
整体图层关系如下：
![](https://i.imgur.com/M4FWuJq.png)


```
SNew(SBox)
			.WidthOverride(500.f)
		.HeightOverride(300.f)
		[

			SNew(SOverlay)

			+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		[
			SNew(SImage)
			.Image(&MenuStyle->GameOptionBGBrush)
		]

	+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		.Padding(FMargin(50.f))
		[
			SNew(SVerticalBox)
	//第一行
		+SVerticalBox::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		.FillHeight(1.f)
		[

		]
	//第二行
	+ SVerticalBox::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		.FillHeight(1.f)
		[
	
		]
	//第三行
	+ SVerticalBox::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		.FillHeight(1.f)
		[
	
		]
```
在第一行中新建水平布局`SHorizontalBox`两列，用于显示切换语言的组件
```
		SNew(SHorizontalBox)
//第一列
			+ SHorizontalBox::Slot()
		.HAlign(HAlign_Center)
		.VAlign(VAlign_Center)
		.FillWidth(1.f)
		[

		]
//第二列
			+ SHorizontalBox::Slot()
		.HAlign(HAlign_Center)
		.VAlign(VAlign_Center)
		.FillWidth(1.f)
		[

		]
```
在`.h`文件中新建指针变量，用来获取`CheckBox`指针和音乐音效的指针
```
private:
//获取CheckBox指针
TSharedPtr<SCheckBox> EnCheckBox;//英文
TSharedPtr<SCheckBox> ZhCheckBox;//中文
//两个进度条
TSharedPtr<SSlider> MuSlider;//背景音乐
TSharedPtr<SSlider> SoSlider;//音效
//进度条百分比
TSharedPtr<STextBlock> MuTextBlock;
TSharedPtr<STextBlock> SoTextBlock;
```
在`.h`文件中创建切换事件
```
private:
//统一设置样式
void StyleInitialize();
//中文CheckBox事件
void ZhCheckBoxStateChanged(ECheckBoxState NewState);
//英文CheckBox事件
void EnCheckBoxStateChanged(ECheckBoxState NewState);
//音量变化事件
void MusicSliderChanged(float Value);
void SoundSliderChanged(float Value);
```

在第一列中实例化第一个`CheckBox`和对应的文本
```
SAssignNew(ZhCheckBox, SCheckBox)
.OnCheckStateChanged(this, &SSlAiGameOptionWidget::ZhCheckBoxStateChanged)
[
SNew(STextBlock)
.Font(MenuStyle->Font_40)
.ColorAndOpacity(MenuStyle->Font_Black)
.Text(NSLOCTEXT("SlAiMenu", "Chinese", "Chinese"))
]
```
同样在第二列实例化英文CheckBox
```
SAssignNew(EnCheckBox, SCheckBox)
.OnCheckStateChanged(this, &SSlAiGameOptionWidget::EnCheckBoxStateChanged)
[
SNew(STextBlock)
.Font(MenuStyle->Font_40)
.ColorAndOpacity(MenuStyle->FontColor_Black)
.Text(NSLOCTEXT("SlAiMenu", "English", "English"))
]
```
在第二行中创建音乐音量调节的组件
```
SNew(SOverlay)
//一号插槽
+ SOverlay::Slot()
.HAlign(HAlign_Left)
.VAlign(VAlign_Center)
[
SNew(STextBlock)
.Font(MenuStyle->Font_40)
.ColorAndOpacity(MenuStyle->FontColor_Black)
.Text(NSLOCTEXT("SlAiMenu", "Music", "Music"))
]
//二号插槽
+ SOverlay::Slot()
.HAlign(HAlign_Center)
.VAlign(VAlign_Center)
[
	SNew(SBox)
	.WidthOverride(240.f)
	[
		SNew(SOverlay)
		+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Center)
		.Padding(FMargin(30.f, 0.f))
		[
		SNew(SImage)
		.Image(&MenuStyle->SliderBarBrush)
		]
		+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Center)
		[
		SAssignNew(MuSlider, SSlider)
		.Style(&MenuStyle->SliderStyle)
		.OnValueChanged(this, &SSlAiGameOptionWidget::MusicSliderChanged)
		]
	
	]
]
//三号插槽
+ SOverlay::Slot()
.HAlign(HAlign_Right)
.VAlign(VAlign_Center)
[
SAssignNew(MuTextBlock, STextBlock)
.Font(MenuStyle->Font_40)
.ColorAndOpacity(MenuStyle->FontColor_Black)
]
```
同理第三行是音效调节（结构同第二行的音量组件）
```
SNew(SOverlay)
+ SOverlay::Slot()
.HAlign(HAlign_Left)
.VAlign(VAlign_Center)
[
SNew(STextBlock)
.Font(MenuStyle->Font_40)
.ColorAndOpacity(MenuStyle->FontColor_Black)
.Text(NSLOCTEXT("SlAiMenu", "Sound", "Sound"))
]
+ SOverlay::Slot()
.HAlign(HAlign_Center)
.VAlign(VAlign_Center)
[
	SNew(SBox)
	.WidthOverride(240.f)
	[
		SNew(SOverlay)
		+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Center)
		.Padding(FMargin(30.f, 0.f))
		[
		SNew(SImage)
		.Image(&MenuStyle->SliderBarBrush)
		]
		+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Center)
		[
		SAssignNew(SoSlider, SSlider)
		.Style(&MenuStyle->SliderStyle)
		.OnValueChanged(this, &SSlAiGameOptionWidget::SoundSliderChanged)
		]
	
	]
]
+ SOverlay::Slot()
.HAlign(HAlign_Right)
.VAlign(VAlign_Center)
[
SAssignNew(SoTextBlock, STextBlock)
.Font(MenuStyle->Font_40)
.ColorAndOpacity(MenuStyle->FontColor_Black)
]
```
为了切换地图后可以延用之前做过的修改，所以将音量变量也放到`SlAiDataHandle`文件下
然后再创建音量控制函数
```
public:
//修改菜单音量
void ResetMenuVolum(float MusicVol, float SoundVol);
//保存音量状态
float MusicVolum;
float SoundVolum;
```
然后实例化控制函数，因为一般只需要对一个变量进行赋值，所以设定不需要赋值时传入负数
```
if (MusicVol > 0)
{
	MusicVolume = MusicVol;
}
if (SoundVol > 0)
{
	SoundVolume = SoundVol;
}
```

虽然创建了相应的组件但是我们需要统一设置其样式
在实例化`StyleInitialize`时建议放在构造函数下方，比较整洁
![](https://i.imgur.com/r4haWME.png)
实例化内容：
```
//设置ZhCheckBox样式
ZhCheckBox->SetUncheckedImage(&MenuStyle->UnCheckedBoxBrush);
ZhCheckBox->SetUncheckedHoveredImage(&MenuStyle->UnCheckedBoxBrush);
ZhCheckBox->SetUncheckedPressedImage(&MenuStyle->UnCheckedBoxBrush);
ZhCheckBox->SetCheckedImage(&MenuStyle->CheckedBoxBrush);
ZhCheckBox->SetCheckedHoveredImage(&MenuStyle->CheckedBoxBrush);
ZhCheckBox->SetCheckedPressedImage(&MenuStyle->CheckedBoxBrush);
//设置EnCheckBox样式
EnCheckBox->SetUncheckedImage(&MenuStyle->UnCheckedBoxBrush);
EnCheckBox->SetUncheckedHoveredImage(&MenuStyle->UnCheckedBoxBrush);
EnCheckBox->SetUncheckedPressedImage(&MenuStyle->UnCheckedBoxBrush);
EnCheckBox->SetCheckedImage(&MenuStyle->CheckedBoxBrush);
EnCheckBox->SetCheckedHoveredImage(&MenuStyle->CheckedBoxBrush);
EnCheckBox->SetCheckedPressedImage(&MenuStyle->CheckedBoxBrush);
//设置滑动条默认位置
MuSlider->SetValue(SlAiDataHandle::Get()->MusicVolume);
SoSlider->SetValue(SlAiDataHandle::Get()->SoundVolume);
//设置音量文本默认显示
MuTextBlock->SetText(FText::FromString(FString::FromInt(FMath::RoundToInt(SlAiDataHandle::Get()->MusicVolume * 100)) + FString("%")));
SoTextBlock->SetText(FText::FromString(FString::FromInt(FMath::RoundToInt(SlAiDataHandle::Get()->SoundVolume * 100)) + FString("%")));

```
然后在`DateHandle.cpp`的构造函数中初始化一下`CurrrentCulture`和`Music/SoundVolume`
```
CurrentCulture = ECultureTeam::ZH;
MusicVolume = 0.5f;
SoundVolume = 0.5f;
```
![](https://i.imgur.com/D1hKCHG.png)
在`StyleInitialize（）`下创建一个`switch`根据`CurrentCulture`判断当前`CheckBox`的选中状态
```
switch (SlAiDataHandle::Get()->CurrentCulture)
{
	case ECultureTeam::EN:
		EnCheckBox->SetIsChecked(ECheckBoxState::Checked);
		ZhCheckBox->SetIsChecked(ECheckBoxState::Unchecked);
		break;
	case ECultureTeam::ZH:
		EnCheckBox->SetIsChecked(ECheckBoxState::Unchecked);
		ZhCheckBox->SetIsChecked(ECheckBoxState::Checked);
		break;
}
```
为了以后可以在多个地图内重复调用更改语言和音乐的函数，我们在`SSlAiGameOptionWidget.h`中声明两个委托
*有关委托的相关知识，详见官方文档或http://www.voidcn.com/article/p-nldddgly-pb.html*
在声明委托前先创建三个类
```
class SCheckBox;
class SSlider;
class STextBlock;
```
这样可以避免不必要的编译错误
然后在类后添加委托声明
```
//修改中英文委托
DECLARE_DELEGATE_OneParam(FChangeCulture, const ECultureTeam)
//修改音量委托
DECLARE_DELEGATE_TwoParams(FChangeVolume, const float, const float)
```
之后就可以在`SLATE_BEGIN_ARGS`后添加委托事件了
```
SLATE_EVENT(FChangeCulture, ChangeCulture)
SLATE_EVENT(FChangeVolume, ChangeVolume
```
最后声明委托的变量
```
private:
FChangeCulture ChangeCulture;
FChangeVolume ChangeVolume;
```
![](https://i.imgur.com/ylQ1eV6.png)

因为是在`MenuWidget`中重复调用，所以在`.h`中声明这两个函数
```
private:
void ChangeCulture(ECultureTeam Culture);
void ChangeVolume(const float MusicVolume, const float SoundVolume);
```

实例化：
![](https://i.imgur.com/GmV8Mmj.png)

并且在ChildSlot中绑定对应事件

![](https://i.imgur.com/1E7uq3n.png)

切换到`SlAiGameOptionWdiget.cpp`
在`ZhCheckBoxStateChanged`中通过委托修改中英文切换
```
EnCheckBox->SetIsChecked(ECheckBoxState::Unchecked);
ZhCheckBox->SetIsChecked(ECheckBoxState::Checked);
ChangeCulture.ExecuteIfBound(ECultureTeam::ZH);
```
英文修改同理

![](https://i.imgur.com/waYAY2k.png)

在`SoundSliderChanged`实例化文本显示和音量修改
```
MuTextBlock->SetText(FText::FromString(FString::FromInt(FMath::RoundToInt(Value * 100)) + FString("%")));
ChangeVolume.ExecuteIfBound(Value, -1.f);
```
音效修改同理
![](https://i.imgur.com/IbaHpZO.png)

编译通过后进入游戏绑定`MenuStyle`
设置字体颜色
![](https://i.imgur.com/voO0cgQ.png)

设置背景和`CheckBox`

![](https://i.imgur.com/CV1her1.png)

设置`Slider`背景和按钮

![](https://i.imgur.com/QcFdvos.png)

最终效果：

![](https://i.imgur.com/QEKxSSp.png)
百分比是根据滑动条实时改变的