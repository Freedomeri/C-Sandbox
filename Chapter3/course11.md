#视角切换与Montage播放
##11.1 实现视角切换功能

1.**添加Input事件**

在ue4编辑器下添加输入按键ChangView——F键，LeftEvent——左键，RightEvent——右键
并在VS里删除`SlAiThirdPlayerAnim.cpp`中`UpdateParameter`函数中的Debug部分

2.**添加PlayerController功能**
重写`Tick` ,`SetupInputComponent` ,`BeginPlay`功能
添加切换视角函数和鼠标按键
在`SlAiPlayerController.h`中声明

```
public:
	ASlAiPlayerController();
	virtual void Tick(float DeltaSeconds) override;
	virtual void SetupInputComponent() override;
public:
	//获取玩家角色
	class ASlAiPlayerCharacter* SPCharacter;
protected:
	virtual void BeginPlay() override;
private:
	//切换视角
	void ChangeView();
	//鼠标按键事件
	void LeftEventStart();
	void LeftEventStop();
	void RightEventStart();
	void RightEventStop();
```
![](https://i.imgur.com/vEO62NN.png)

在`SlAiPlayerController.cpp`文件引入头文件
```
#include "SlAiPlayerCharacter.h"
```
在实例化功能前，先在`SlAiTypes.h`定义视角模式的枚举类

![](https://i.imgur.com/jpcRnML.png)

在`SlAiPlayerController.h`和`SlAiPlayerCharacter.h`引入头文件
```
#include "SlAiTypes.h"
```
并在`SlAiPlayerCharacter.h`添加`Public`变量

![](https://i.imgur.com/DSK1lNi.png)

并在`cpp`文件中初始化为第三人称

![](https://i.imgur.com/vl8NjiH.png)

在`SlAiPlayerCharacter.h`重新声明`Public`函数`void ChangeView(EGameViewMode::Type NewGameView);`
实例化：
```
void ASlAiPlayerCharacter::ChangeView(EGameViewMode::Type NewGameView)
{
	GameView = NewGameView;
	switch (GameView)
	{
	case EGameViewMode::First:
		FirstCamera->SetActive(true);
		ThirdCamera->SetActive(false);
		MeshFirst->SetOwnerNoSee(false);
		GetMesh()->SetOwnerNoSee(true);
		
		break;
	case EGameViewMode::Third:
		FirstCamera->SetActive(false);
		ThirdCamera->SetActive(true);
		MeshFirst->SetOwnerNoSee(true);
		GetMesh()->SetOwnerNoSee(false);
		break;
	}
}
```




`SlAiPlayerController.cpp`实例化函数：

```
ASlAiPlayerController::ASlAiPlayerController()
{
	//允许每帧运行
	PrimaryActorTick.bCanEverTick = true;
}
void ASlAiPlayerController::BeginPlay()
{
	Super::BeginPlay();
	//获取角色与状态
	if (!SPCharacter) SPCharacter = Cast<ASlAiPlayerCharacter>(GetCharacter());
	if (!SPState) SPState = Cast<ASlAiPlayerState>(PlayerState);
	//设置鼠标不显示
	bShowMouseCursor = false;
	//设置输入模式
	FInputModeGameOnly InputMode;
	InputMode.SetConsumeCaptureMouseDown(true);
	SetInputMode(InputMode);
}
void ASlAiPlayerController::Tick(float DeltaSeconds)
{

}
void ASlAiPlayerController::SetupInputComponent()
{
	Super::SetupInputComponent();
	//绑定视角切换
	InputComponent->BindAction("ChangeView", IE_Pressed, this, &ASlAiPlayerController::ChangeView);
	//绑定鼠标按下事件
	InputComponent->BindAction("LeftEvent", IE_Pressed, this, &ASlAiPlayerController::LeftEventStart);
	InputComponent->BindAction("LeftEvent", IE_Released, this, &ASlAiPlayerController::LeftEventStop);
	InputComponent->BindAction("RightEvent", IE_Pressed, this, &ASlAiPlayerController::RightEventStart);
	InputComponent->BindAction("RightEvent", IE_Released, this, &ASlAiPlayerController::RightEventStop);
}
void ASlAiPlayerController::ChangeView()
{
	
	switch (SPCharacter->GameView)
	{
	case EGameViewMode::First:
		SPCharacter->ChangeView(EGameViewMode::Third);
		break;
	case EGameViewMode::Third:
		SPCharacter->ChangeView(EGameViewMode::First);
		break;
	}
}
```
编译运行通过，进入编辑器就可以实现视角切换

##11.2 Montage播放
1.**设置动作蓝图**
在`Content\Blueprint\Player`文件下有绑定好的Montage动作，打开后可以预览动作
打开`ThirdPlayer_Animation`进入蓝图编辑器
然后编辑蓝图：

![](https://i.imgur.com/OljEIED.png)

骨骼混合设置：

![](https://i.imgur.com/7sA9Xzz.png)

上半身锁定在视角的正前方，移动时不应该旋转太大
设置：

![](https://i.imgur.com/xFzSJPe.png)

**同样的设置也要在第一人称的动作上设置一次**
*第一人称动作文件`FirstPlayer_Animation`  *
*有关Montage动作和蓝图控制详见官方文档*

在`SlAiPlayerAnim`中`UpdateParameter()`函数设置绑定的参数

![](https://i.imgur.com/bYSlI92.png)

2.**设置动作的指针**
在`SlAiPlayerAnim.h`设置动作的指针
```
protected:
//上半身的Montage
	UAnimMontage* PlayerHitMontage;
	UAnimMontage* PlayerFightMontage;
	UAnimMontage* PlayerPunchMontage;
	UAnimMontage* PlayerEatMontage;
	UAnimMontage* PlayerPickUpMontage;
```
并引入头文件
```
#include "Animation/AnimMontage.h"
```
且声明`UAnimMontage`(不声明无法通过编译)

![](https://i.imgur.com/YJpuzzL.png)

在`SlAiThirdPlayerAnim.cpp`的构造函数中加入
```
	//绑定资源到Montage
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerHitMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/Player/Animation/UpperBody/PlayerHitMontage.PlayerHitMontage'"));
	PlayerHitMontage = PlayerHitMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerEatMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/Player/Animation/UpperBody/PlayerEatMontage.PlayerEatMontage'"));
	PlayerEatMontage = PlayerEatMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerFightMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/Player/Animation/UpperBody/PlayerFightMontage.PlayerFightMontage'"));
	PlayerFightMontage = PlayerFightMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerPunchMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/Player/Animation/UpperBody/PlayerPunchMontage.PlayerPunchMontage'"));
	PlayerPunchMontage = PlayerPunchMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerPickUpMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/Player/Animation/UpperBody/PlayerPickUpMontage.PlayerPickUpMontage'"));
	PlayerPickUpMontage = PlayerPickUpMon.Object;
	//设置自己的运行人称是第三人称
	GameView = EGameViewMode::Third;
```
顺便加入头文件
```
#include "ConstructorHelpers.h"
#include "Animation/AnimMontage.h"
```

在`SlAiFirstPlayerAnim.h`声明构造函数

![](https://i.imgur.com/QuH9YSZ.png)

在`SlAiFirstPlayerAnim.cpp`初始化
```
//绑定资源到Montage
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerHitMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/FirstPlayer/Animation/UpperBody/FirstPlayerHitMontage.FirstPlayerHitMontage'"));
	PlayerHitMontage = PlayerHitMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerEatMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/FirstPlayer/Animation/UpperBody/FirstPlayerEatMontage.FirstPlayerEatMontage'"));
	PlayerEatMontage = PlayerEatMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerFightMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/FirstPlayer/Animation/UpperBody/FirstPlayerFightMontage.FirstPlayerFightMontage'"));
	PlayerFightMontage = PlayerFightMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerPunchMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/FirstPlayer/Animation/UpperBody/FirstPlayerPunchMontage.FirstPlayerPunchMontage'"));
	PlayerPunchMontage = PlayerPunchMon.Object;
	static ConstructorHelpers::FObjectFinder<UAnimMontage> PlayerPickUpMon(TEXT("AnimMontage'/Game/Res/PolygonAdventure/Mannequin/FirstPlayer/Animation/UpperBody/FirstPlayerPickUpMontage.FirstPlayerPickUpMontage'"));
	PlayerPickUpMontage = PlayerPickUpMon.Object;
	//设置自己的运行人称是第一人称
	GameView = EGameViewMode::First;
```
并引入头文件
```
#include "ConstructorHelpers.h"
#include "Animation/AnimMontage.h"
```
3.**播放动作**
在`SlAiPlayerAnim.h`中声明`protected`函数`virtual void UpdateMontage();`

实例化：
在`NativeUpdateAnimation(float DeltaSeconds)`函数下添加
```
//更新动作
	UpdateMontage();
//修改是否允许切换视角
	void AllowViewChange(bool IsAllow);
```
回到`SlAiTypes.h`中添加枚举类

![](https://i.imgur.com/6qs6Xnb.png)

在`SlAiPlayerCharacter.h`添加`public`类型变量
```
//上半身动画状态
	EUpperBody::Type UpperType;
```

并在构造函数初始化为无动作
```
	//上半身动作初始为无动作
	UpperType = EUpperBody::None;
```
在`SlAiPlayerAnim.h`添加`protected`类型变量
```
	//保存当前播放的Montage
	UAnimMontage* CurrentMontage;
	//指定自己的运行人称
	EGameViewMode::Type GameView;
```
并引用数据结构类`#include "SlAiTypes.h"`

在`SlAiPlayerCharacter.h`声明`public`类型
```
	//是否允许切换视角
	bool IsAllowSwitch;
```
并在构造函数初始化
```
	//一开始允许切换视角
	IsAllowSwitch = true;
```
在`SlAiPlayerController.cpp`中`ChangeView（）`在`switch`添加判断
```
	//如果不允许切换视角,直接返回
	if (!SPCharacter->IsAllowSwitch) return;
```
实例化`void AllowViewChange(bool IsAllow);`
```
	if (!SPCharacter) return;
	SPCharacter->IsAllowSwitch = IsAllow;
```

实例化`UpdateMontage()`函数
```
//如果不存在直接返回,避免空指针产生中断
	if (!SPCharacter) return;
	//如果当前的人称状态和这个动作的不一致,直接返回
	if (SPCharacter->GameView != GameView) return;
	//如果当前的动作没有停止,不更新动作
	if (!Montage_GetIsStopped(CurrentMontage)) return;
	switch (SPCharacter->UpperType)
	{
	case EUpperBody::None:
		//如果有哪个动作在播放
		if (CurrentMontage != nullptr) {
			Montage_Stop(0);
			CurrentMontage = nullptr;
			//允许切换视角
			AllowViewChange(true);
		}
		break;
	case EUpperBody::Punch:
		if (!Montage_IsPlaying(PlayerPunchMontage)) {
			Montage_Play(PlayerPunchMontage);
			CurrentMontage = PlayerPunchMontage;
			//不允许切换视角
			AllowViewChange(false);
		}
		break;
	case EUpperBody::Hit:
		if (!Montage_IsPlaying(PlayerHitMontage)) {
			Montage_Play(PlayerHitMontage);
			CurrentMontage = PlayerHitMontage;
			AllowViewChange(false);
		}
		break;
	case EUpperBody::Fight:
		if (!Montage_IsPlaying(PlayerFightMontage)) {
			Montage_Play(PlayerFightMontage);
			CurrentMontage = PlayerFightMontage;
			AllowViewChange(false);
		}
		break;
	case EUpperBody::PickUp:
		if (!Montage_IsPlaying(PlayerPickUpMontage)) {
			Montage_Play(PlayerPickUpMontage);
			CurrentMontage = PlayerPickUpMontage;
			AllowViewChange(false);
		}
		break;
	case EUpperBody::Eat:
		if (!Montage_IsPlaying(PlayerEatMontage)) {
			Montage_Play(PlayerEatMontage);
			CurrentMontage = PlayerEatMontage;
			AllowViewChange(false);
		}
		break;
	}

```
在`SlAiPlayerController.h`添加`private`变量
```
	//左键预动作
	EUpperBody::Type LeftUpperType;
	//右键预动作
	EUpperBody::Type RightUpperType;

```
在`BeginPlay`下初始化预动作
```
	//设置预动作
	LeftUpperType = EUpperBody::Punch;
	RightUpperType = EUpperBody::PickUp;
```
并写完未实例化的鼠标事件：
```
void ASlAiPlayerController::LeftEventStart()
{
	SPCharacter->UpperType = LeftUpperType;
}
void ASlAiPlayerController::LeftEventStop()
{
	SPCharacter->UpperType = EUpperBody::None;
}
void ASlAiPlayerController::RightEventStart()
{
	SPCharacter->UpperType = RightUpperType;
}
void ASlAiPlayerController::RightEventStop()
{
	SPCharacter->UpperType = EUpperBody::None;
}
```
此时编译通过就可以看到人物播放动作且在播放的时候不会因为切换视角而重复触发动作
且人物上半身一直面朝视角方向