#小地图玩家与敌人
##29.1 小地图玩家方位
1.**事先准备**
在项目设置中新建按键

![](https://i.imgur.com/70BUP2n.png)

在数据结构类添加新的数据结构

![](https://i.imgur.com/D0rhgij.png)

2.**功能实现**
在`SlAiSceneCapture2D`头文件声明：
```
public:
	//更新视野范围
	void UpdateMiniMapWidth(int Delta);
	//获取小地图尺寸
	float GetMapSize();
```
实现更新视野范围：
```
	const float PreWidth = GetCaptureComponent2D()->OrthoWidth;
	GetCaptureComponent2D()->OrthoWidth = FMath::Clamp<float>(PreWidth + Delta, 2000.f, 4000.f);
```
实现获得小地图的大小
```
	return GetCaptureComponent2D()->OrthoWidth;
```
在`SlAiPlayerController`头文件下声明委托：
```
//修改小地图视野范围委托
DECLARE_DELEGATE_OneParam(FUpdateMiniMapWidth, int)
public:
	//修改小地图视野范围委托,注册函数是SlAiSceneCapture2D的UpdateMiniMapWidth 
	FUpdateMiniMapWidth UpdateMiniMapWidth;
private:
	//小地图缩放事件
	void AddMapSizeStart();
	void AddMapSizeStop();
	void ReduceMapSizeStart();
	void ReduceMapSizeStop();
	//在Tick函数处理小地图事件
	void TickMiniMap();
	//小地图缩放状态
	EMiniMapSizeMode::Type MiniMapSizeMode;
```
在`BeginPlay`函数初始化缩放状态
```
	//小地图缩放状态
	EMiniMapSizeMode::Type MiniMapSizeMode;
```
在`SetupInputComponent`函数绑定事件
```
	//绑定缩放小地图事件
	InputComponent->BindAction("AddMapSize", IE_Pressed, this, &ASlAiPlayerController::AddMapSizeStart);
	InputComponent->BindAction("AddMapSize", IE_Released, this, &ASlAiPlayerController::AddMapSizeStop);
	InputComponent->BindAction("ReduceMapSize", IE_Pressed, this, &ASlAiPlayerController::ReduceMapSizeStart);
	InputComponent->BindAction("ReduceMapSize", IE_Released, this, &ASlAiPlayerController::ReduceMapSizeStop);
```
实现小地图缩放事件：
```
void ASlAiPlayerController::AddMapSizeStart()
{
	//如果操作被锁住,直接返回
	if (SPCharacter->IsInputLocked) return;
	//设置缩放状态为增加
	MiniMapSizeMode = EMiniMapSizeMode::Add;
}

void ASlAiPlayerController::AddMapSizeStop()
{
	//如果操作被锁住,直接返回
	if (SPCharacter->IsInputLocked) return;
	//设置缩放状态为无
	MiniMapSizeMode = EMiniMapSizeMode::None;
}

void ASlAiPlayerController::ReduceMapSizeStart()
{
	//如果操作被锁住,直接返回
	if (SPCharacter->IsInputLocked) return;
	//设置缩放状态为减少
	MiniMapSizeMode = EMiniMapSizeMode::Reduce;
}

void ASlAiPlayerController::ReduceMapSizeStop()
{
	//如果操作被锁住,直接返回
	if (SPCharacter->IsInputLocked) return;
	//设置缩放状态为无
	MiniMapSizeMode = EMiniMapSizeMode::None;
}
```
然后把`TickMiniMap`函数放在Tick函数下
实现：
```
	switch (MiniMapSizeMode)
	{
	case EMiniMapSizeMode::Add:
		UpdateMiniMapWidth.ExecuteIfBound(5);
		break;
	case EMiniMapSizeMode::Reduce:
		UpdateMiniMapWidth.ExecuteIfBound(-5);
		break;
	}
```
然后到`SlAiGameMode`进行绑定

![](https://i.imgur.com/dv4T5LK.png)

再到头文件去声明一个委托
```
//更新MiniMap的数据
DECLARE_DELEGATE_FiveParams(FUpdateMapData, const FRotator, const float, const TArray<FVector2D>*, const TArray<bool>*, const TArray<float>*)
public:
	//定义委托,用于更新小地图的方向文字位置,绑定的方法的MiniMapWidget的UpdateMapDirection
	FUpdateMapData UpdateMapData;
```
在`InitializeMiniMapCamera`函数里添加代码：

![](https://i.imgur.com/vZVe6EG.png)

再到`SlAiGameHUD.cpp`绑定委托
在`BeginPlay`下：
```
	//绑定更新小地图数据委托
	GM->UpdateMapData.BindRaw(GameHUDWidget->MiniMapWidget.Get(), &SSlAiMiniMapWidget::UpdateMapData);
```

在`SlAiMiniMapWidget`头文件声明：
```
public:
	//委托接受GameMode传过来的玩家旋转,绑定的委托是GameMode的UpdateMapDirection
	void UpdateMapData(const FRotator PlayerRotator, const float MiniMapSize, const TArray<FVector2D>* EnemyPosList, const TArray<bool>* EnemyLockList, const TArray<float>* EnemyRotateList);
	//重写绘制函数
	virtual int32 OnPaint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const override;

private:
	//四个方向的渲染位置
	FVector2D NorthLocation;
	FVector2D SouthLocation;
	FVector2D EastLocation;
	FVector2D WestLocation;
```
实现`UpdateMapData`函数
```
	//获取Yaw,这个Yaw从180到-180,我们把他变成负的,然后通过加角度来计算
	float YawDir = -PlayerRotator.Yaw;
	//使用三角函数来计算
	NorthLocation = FVector2D(FMath::Sin(FMath::DegreesToRadians(YawDir)), FMath::Cos(FMath::DegreesToRadians(YawDir))) * 150.f + FVector2D(160.f, 160.f);
	EastLocation = FVector2D(FMath::Sin(FMath::DegreesToRadians(YawDir + 90.f)), FMath::Cos(FMath::DegreesToRadians(YawDir + 90.f)))*150.f + FVector2D(160.f, 160.f);
	SouthLocation = FVector2D(FMath::Sin(FMath::DegreesToRadians(YawDir + 180.f)), FMath::Cos(FMath::DegreesToRadians(YawDir + 180.f)))*150.f + FVector2D(160.f, 160.f);
	WestLocation = FVector2D(FMath::Sin(FMath::DegreesToRadians(YawDir + 270.f)), FMath::Cos(FMath::DegreesToRadians(YawDir + 270.f)))*150.f + FVector2D(160.f, 160.f);

```
实现`OnPaint`函数：
```
	//先调用一下父类函数
	SCompoundWidget::OnPaint(Args, AllottedGeometry, MyCullingRect, OutDrawElements, LayerId, InWidgetStyle, bParentEnabled);
	//渲染玩家图标
	FSlateDrawElement::MakeBox(
		OutDrawElements,
		LayerId + 10,
		AllottedGeometry.ToPaintGeometry(FVector2D(155.f, 155.f), FVector2D(10.f, 10.f)),
		&GameStyle->PawnPointBrush,
		ESlateDrawEffect::None,
		FLinearColor(1.f, 1.f, 0.f, 1.f)
	);
	//渲染东西南北文字
	FSlateDrawElement::MakeText(
		OutDrawElements,
		LayerId + 10,
		AllottedGeometry.ToPaintGeometry(NorthLocation - FVector2D(8.f, 8.f), FVector2D(16.f, 16.f)),
		NSLOCTEXT("SlAiGame", "N", "N"),
		GameStyle->Font_16,
		ESlateDrawEffect::None,
		FLinearColor(1.f, 1.f, 1.f, 1.f)
	);
	FSlateDrawElement::MakeText(
		OutDrawElements,
		LayerId + 10,
		AllottedGeometry.ToPaintGeometry(SouthLocation - FVector2D(8.f, 8.f), FVector2D(16.f, 16.f)),
		NSLOCTEXT("SlAiGame", "S", "S"),
		GameStyle->Font_16,
		ESlateDrawEffect::None,
		FLinearColor(1.f, 1.f, 1.f, 1.f)
	);
	FSlateDrawElement::MakeText(
		OutDrawElements,
		LayerId + 10,
		AllottedGeometry.ToPaintGeometry(EastLocation - FVector2D(8.f, 8.f), FVector2D(16.f, 16.f)),
		NSLOCTEXT("SlAiGame", "E", "E"),
		GameStyle->Font_16,
		ESlateDrawEffect::None,
		FLinearColor(1.f, 1.f, 1.f, 1.f)
	);
	FSlateDrawElement::MakeText(
		OutDrawElements,
		LayerId + 10,
		AllottedGeometry.ToPaintGeometry(WestLocation - FVector2D(8.f, 8.f), FVector2D(16.f, 16.f)),
		NSLOCTEXT("SlAiGame", "W", "W"),
		GameStyle->Font_16,
		ESlateDrawEffect::None,
		FLinearColor(1.f, 1.f, 1.f, 1.f)
	);
```
然后进入`SlAiInternation.h`添加本地化文字

![](https://i.imgur.com/cHi3nGh.png)

此时可以编译通过
进入编辑器添加玩家图标

![](https://i.imgur.com/mQkGIZk.png)

然后重新识别本地化文字然后依次翻译
此时进入游戏就可以看到小地图的方位旋转和玩家图标

##29.2 小地图敌人视野
1.**准备工作**
新建敌人视野Material组件

![](https://i.imgur.com/Y30zk00.png)

连接节点（材质名称：Enemy View）

![](https://i.imgur.com/vAburYS.png)

新建材质（敌人视野）

![](https://i.imgur.com/Bp5n19O.png)


![](https://i.imgur.com/oWlb95N.png)


![](https://i.imgur.com/DEPHe10.png)

![](https://i.imgur.com/pvzPlLT.png)

创建材质实例（把其中所有的选项全部勾选）

![](https://i.imgur.com/DEuQpzL.png)

2.**功能实现**

在`SlAiGameMode`的`InitializeMiniMapCamera`函数中将代码替换为：
```
	//如果摄像机还不存在并且世界已经存在
	if (!IsCreateMiniMap && GetWorld())
	{
		//生成小地图摄像机
		MiniMapCamera = GetWorld()->SpawnActor<ASlAiSceneCapture2D>(ASlAiSceneCapture2D::StaticClass());
		//运行委托给MiniMapWidget传递渲染的MiniMapTex
		RegisterMiniMap.ExecuteIfBound(MiniMapCamera->GetMiniMapTex());
		//绑定修改小地图视野的委托
		SPController->UpdateMiniMapWidth.BindUObject(MiniMapCamera, &ASlAiSceneCapture2D::UpdateMiniMapWidth);
		//设置已经生成小地图
		IsCreateMiniMap = true;
	}
	//如果小地图已经创建
	if (IsCreateMiniMap)
	{
		//每帧更新小地图摄像机的位置和旋转
		MiniMapCamera->UpdateTransform(SPCharacter->GetActorLocation(), SPCharacter->GetActorRotation());
		TArray<FVector2D> EnemyPosList;
		TArray<bool> EnemyLockList;
		TArray<float> EnemyRotateList;
		//获取场景中的敌人
		for (TActorIterator<ASlAiEnemyCharacter> EnemyIt(GetWorld()); EnemyIt; ++EnemyIt)
		{
			FVector EnemyPos = FVector((*EnemyIt)->GetActorLocation().X - SPCharacter->GetActorLocation().X, (*EnemyIt)->GetActorLocation().Y - SPCharacter->GetActorLocation().Y, 0.f);
			EnemyPos = FQuat(FVector::UpVector, FMath::DegreesToRadians(-SPCharacter->GetActorRotation().Yaw - 90.f)) * EnemyPos;
			EnemyPosList.Add(FVector2D(EnemyPos.X, EnemyPos.Y));
			EnemyLockList.Add((*EnemyIt)->IsLockPlayer());
			EnemyRotateList.Add((*EnemyIt)->GetActorRotation().Yaw - SPCharacter->GetActorRotation().Yaw);
		}
		//每帧更新小地图的方向文字位置
		UpdateMapData.ExecuteIfBound(SPCharacter->GetActorRotation(), MiniMapCamera->GetMapSize(), &EnemyPosList, &EnemyLockList, &EnemyRotateList);
```
在`SlAiEnemyCharacter.h`声明public函数
```
	//获取是否已经锁定了玩家
	bool IsLockPlayer();
```
实现：
```
	if (SEController) return SEController->IsLockPlayer;
	return false;
```
在`SlAiMiniMapWidget.h`声明private:
```
	//小地图尺寸
	float MapSize;
	//敌人相对于玩家的位置
	TArray<FVector2D> EnemyPos;
	//敌人是否锁定了玩家
	TArray<bool> EnemyLock;
	//显示玩家视野的图片
	TSharedPtr<SImage> EnemyViewImage;
```
在cpp文件添加组件

![](https://i.imgur.com/whOcyuw.png)

在`RegisterMiniMap`函数添加：
```
	//敌人视野材质设定
	UMaterialInterface* EnemyViewMatInst = LoadObject<UMaterialInterface>(NULL, TEXT("MaterialInstanceConstant'/Game/Material/EnemyViewMatInst.EnemyViewMatInst'"));
	//创建动态材质
	EnemyViewMatDynamic = UMaterialInstanceDynamic::Create(EnemyViewMatInst, nullptr);
	//实例化EnemyView笔刷
	FSlateBrush* EnemyViewBrush = new FSlateBrush();
	//设置属性
	EnemyViewBrush->ImageSize = FVector2D(280.f, 280.f);
	EnemyViewBrush->DrawAs = ESlateBrushDrawType::Image;
	//绑定材质资源文件
	EnemyViewBrush->SetResourceObject(EnemyViewMatDynamic);
	//将笔刷作为MiniMapImage的笔刷
	EnemyViewImage->SetImage(EnemyViewBrush);
	//颜色为透明绿
	EnemyViewImage->SetColorAndOpacity(FLinearColor(0.3f, 1.f, 0.32f, 0.4f));
```
在`UpdateMapData`添加代码：
```
	//地图尺寸
	MapSize = MiniMapSize;
	//清空现在的敌人列表
	EnemyPos.Empty();
	//清空敌人是否锁定列表
	EnemyLock.Empty();
	//比例
	float DPIRatio = 280.f / MapSize;
	//保存视野旋转信息
	TArray<float> EnemyViewRotate;
	//保存视野位置信息
	TArray<FVector2D> EnemyViewPos;
	//保存视野锁定信息
	TArray<bool> EnemyViewLock;
	//获取敌人信息
	for (int i = 0; i < (*EnemyPosList).Num(); ++i) {
		//计算实际长度
		float RealDistance = (*EnemyPosList)[i].Size();
		//如果长度小于地图实际半径
		if (RealDistance * 2 < MapSize)
		{
			//屏幕位置
			EnemyPos.Add((*EnemyPosList)[i] * DPIRatio + FVector2D(160.f, 160.f));
			//是否锁定玩家
			EnemyLock.Add((*EnemyLockList)[i]);
		}
		//如果长度小于地图实际半径再加上2000,就渲染到视野
		if (RealDistance * 2 < MapSize + 2000.f)
		{
			//屏幕位置
			EnemyViewPos.Add((*EnemyPosList)[i] * DPIRatio + FVector2D(160.f, 160.f));
			//是否锁定玩家
			EnemyViewLock.Add((*EnemyLockList)[i]);
			//添加旋转信息,格式化为0-1
			float RotVal = -(*EnemyRotateList)[i];
			if (RotVal > 180.f) RotVal -= 360.f;
			if (RotVal < -180.f) RotVal += 360.f;
			//序列化到0-360
			RotVal += 180.f;
			//序列化0-1
			RotVal /= 360.f;
			//转个180度
			RotVal = RotVal + 0.5f > 1.f ? RotVal - 0.5f : RotVal + 0.5f;
			//天际进数组
			EnemyViewRotate.Add(RotVal);
		}
	}
	int ViewCount = 0;
	//修改敌人视野缩放比例
	EnemyViewMatDynamic->SetScalarParameterValue(FName("Scale"), 1000.f / MapSize);
	for (int i = 0; i < EnemyViewPos.Num(); ++i, ++ViewCount) {
		FString PosName = FString("Position_") + FString::FromInt(i + 1);
		FString AngleName = FString("Angle_") + FString::FromInt(i + 1);
		//如果没锁定玩家就渲染
		if (!EnemyViewLock[i]) {
			EnemyViewMatDynamic->SetVectorParameterValue(FName(*PosName), FLinearColor((EnemyViewPos[i].X - 20.f) / 280.f, (EnemyViewPos[i].Y - 20.f) / 280.f, 0.f, 0.f));
			EnemyViewMatDynamic->SetScalarParameterValue(FName(*AngleName), EnemyViewRotate[i]);
		}
		else
		{
			EnemyViewMatDynamic->SetVectorParameterValue(FName(*PosName), FLinearColor(0.f, 0.f, 0.f, 0.f));
			EnemyViewMatDynamic->SetScalarParameterValue(FName(*AngleName), 0.f);
		}
	}
	//把剩下的视野都不渲染
	for (ViewCount += 1; ViewCount < 11; ++ViewCount) {
		FString PosName = FString("Position_") + FString::FromInt(ViewCount);
		FString AngleName = FString("Angle_") + FString::FromInt(ViewCount);
		EnemyViewMatDynamic->SetVectorParameterValue(FName(*PosName), FLinearColor(0.f, 0.f, 0.f, 0.f));
		EnemyViewMatDynamic->SetScalarParameterValue(FName(*AngleName), 0.f);
	}
```
编译通过就可以看到小地图显示敌人的位置和视野