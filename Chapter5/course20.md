# 鼠标选中背包容器 #
##20.1 鼠标选中背包容器

1.**Widget获取位置和大小**
`FGeometry`类保存Widget位置和大小变换
`Position`获取Widget相对于父类Widget的位置
`AbsolutePosition`获取Widget在整个电脑屏幕上的位置
`Size`可以获取Widget大小
2.**获取鼠标位置**
在`SlAiPackageWidget.h`声明`private`变量
```
	//鼠标位置标定
	FVector2D MousePosition;
	//DPI的缩放
	TAttribute<float> UIScaler;
	//是否已经初始化背包管理器
	bool IsInitPackageMana;
```
再声明`public`宏
```
	SLATE_ATTRIBUTE(float, UIScaler)
```
并重写`Tick`函数
```
virtual void Tick(const FGeometry& AllottedGeometry, const double InCurrentTime, const float InDeltaTime) override;
```
现在构造函数进行初始化
```
	//获取DPIScaler
	UIScaler = InArgs._UIScaler;
	MousePosition = FVector2D(0.f, 0.f);
	IsInitPackageMana = false;
```
引入头文件
```
#include "Engine/GameEngine.h"
#include "Engine/Engine.h"
#include "SlAiPackageManager.h"
```
`Tick`函数实现：
```
	//如果背包显示并且世界存在,实时更新鼠标位置
	if (GetVisibility() == EVisibility::Visible && GEngine)
	{
		GEngine->GameViewport->GetMousePosition(MousePosition);
		MousePosition = MousePosition / UIScaler.Get();
	}
	//如果背包管理器已经初始化
	if (IsInitPackageMana) {
		//实时更新容器悬停显示
		SlAiPackageManager::Get()->UpdateHovered(MousePosition, AllottedGeometry);
	}
```
在`SlAiGameHUDWidget.cpp`传入`UIScaler`参数

![](https://i.imgur.com/oEm6zMx.png)

3.**实现选中Widget高亮**
在`SlAiPackageManager.h `删除析构函数并声明：
```
public:
	static void Initialize();
	static TSharedPtr<SlAiPackageManager> Get();
	//添加容器
	void InsertContainer(TSharedPtr<class SSlAiContainerBaseWidget> Container, EContainerType::Type InsertType);
	//更新悬停的容器颜色
	void UpdateHovered(FVector2D MousePos, FGeometry PackGeo);
private:
	//创建实例方法
	static TSharedRef<SlAiPackageManager> Create();
	//获取鼠标指向的容器
	TSharedPtr<SSlAiContainerBaseWidget> LocateContainer(FVector2D MousePos, FGeometry PackGeo);
	//单例指针
	static TSharedPtr<SlAiPackageManager> PackageInstance;
	//容器列表
	TArray<TSharedPtr<SSlAiContainerBaseWidget>> InputContainerList;
	TArray<TSharedPtr<SSlAiContainerBaseWidget>> NormalContainerList;
	TArray<TSharedPtr<SSlAiContainerBaseWidget>> ShortcutContainerList;
	//输出容器只有一个
	TSharedPtr<SSlAiContainerBaseWidget> OutputContainer;
	//上一个悬停的容器
	TSharedPtr<SSlAiContainerBaseWidget> LastHoveredCon;
```
引入头文件
```
#include "SlAiTypes.h"
#include "SSlAiContainerBaseWidget.h"
```
单例模式实现：

![](https://i.imgur.com/aN4gitq.png)

`InsertContainer`实现：
```
	switch (InsertType)
	{
	case EContainerType::Output:
		OutputContainer = Container;
		break;
	case EContainerType::Input:
		InputContainerList.Add(Container);
		break;
	case EContainerType::Normal:
		NormalContainerList.Add(Container);
		break;
	case EContainerType::Shortcut:
		ShortcutContainerList.Add(Container);
		break;
	}
```

`LocateContainer`实现：
```
	//迭代找到指向的容器
	for (TArray<TSharedPtr<SSlAiContainerBaseWidget>>::TIterator It(ShortcutContainerList); It; ++It) {
		//获取区域
		FVector2D StartPos = PackGeo.AbsoluteToLocal((*It)->GetCachedGeometry().AbsolutePosition);
		FVector2D EndPos = StartPos + FVector2D(80.f, 80.f);
		//判断鼠标位置是否在区域内,在的话直接返回这个容器
		if (MousePos.X >= StartPos.X && MousePos.X <= EndPos.X && MousePos.Y >= StartPos.Y && MousePos.Y <= EndPos.Y)
		{
			return *It;
		}
	}
	for (TArray<TSharedPtr<SSlAiContainerBaseWidget>>::TIterator It(NormalContainerList); It; ++It) {
		//获取区域
		FVector2D StartPos = PackGeo.AbsoluteToLocal((*It)->GetCachedGeometry().AbsolutePosition);
		FVector2D EndPos = StartPos + FVector2D(80.f, 80.f);
		//判断鼠标位置是否在区域内,在的话直接返回这个容器
		if (MousePos.X >= StartPos.X && MousePos.X <= EndPos.X && MousePos.Y >= StartPos.Y && MousePos.Y <= EndPos.Y) {
			return *It;
		}
	}
	for (TArray<TSharedPtr<SSlAiContainerBaseWidget>>::TIterator It(InputContainerList); It; ++It) {
		//获取区域
		FVector2D StartPos = PackGeo.AbsoluteToLocal((*It)->GetCachedGeometry().AbsolutePosition);
		FVector2D EndPos = StartPos + FVector2D(80.f, 80.f);
		//判断鼠标位置是否在区域内,在的话直接返回这个容器
		if (MousePos.X >= StartPos.X && MousePos.X <= EndPos.X && MousePos.Y >= StartPos.Y && MousePos.Y <= EndPos.Y) {
			return *It;
		}
	}
	//这里处理输出容器的
	//获取区域
	FVector2D StartPos = PackGeo.AbsoluteToLocal(OutputContainer->GetCachedGeometry().AbsolutePosition);
	FVector2D EndPos = StartPos + FVector2D(80.f, 80.f);
	//判断鼠标位置是否在区域内,在的话直接返回这个容器
	if (MousePos.X >= StartPos.X && MousePos.X <= EndPos.X && MousePos.Y >= StartPos.Y && MousePos.Y <= EndPos.Y) {
		return OutputContainer;
	}
	//最后返回空
	return nullptr;
```

在实现更新背景之前先到`SlAiContainerBaseWidget `里实现改变背景颜色的方法
声明：
```
public:
	//更新鼠标移动到上面的状态
	void UpdateHovered(bool IsHovered);
protected:
	//是否在Hover状态
	bool IsHover;
```
然后再构造函数初始化：
```
	IsHover = false;
```
`SlAiContainerBaseWidget`中`UpdateHovered`函数实现：
```
	//如果鼠标移动到上面了
	if (IsHovered) 
	{
		if (!IsHover) ContainerBorder->SetBorderImage(&GameStyle->ChoosedContainerBrush);
	}
	else 
	{
		if (IsHover) ContainerBorder->SetBorderImage(&GameStyle->NormalContainerBrush);
	}
	//更新当前状态
	IsHover = IsHovered;
```


`SlAiPackageManager`中`UpdateHovered`函数实现
```
	//先获取悬停的容器
	TSharedPtr<SSlAiContainerBaseWidget> CurrHoveredCon = LocateContainer(MousePos, PackGeo);
	//如果容器存在
	if (CurrHoveredCon.IsValid())
	{
		//设置当前容器悬停显示
		CurrHoveredCon->UpdateHovered(true);
		//如果上一容器存在,并且与当前容器不相同
		if (LastHoveredCon.IsValid() && LastHoveredCon.Get() != CurrHoveredCon.Get()) {
			//更新悬停显示
			LastHoveredCon->UpdateHovered(false);
		}
	}
	else 
	{
		//当前容器不存在且上一容器存在,取消上一容器的悬停显示
		if (LastHoveredCon.IsValid()) {
			LastHoveredCon->UpdateHovered(false);
		}
	}

	//更新上一悬停容器
	LastHoveredCon = CurrHoveredCon;
```

回到`SlAiPackageWidget`,在`InitPackageManager`添加语句

![](https://i.imgur.com/tbtgzv7.png)
在最后添加:
```
	//设置已经初始化背包管理器
	IsInitPackageMana = true;
```
编译通过就可以看到鼠标悬停在背包栏时会有变红显示

