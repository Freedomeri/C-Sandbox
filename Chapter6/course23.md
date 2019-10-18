#敌人动作与组件
##23.1 敌人模型与动作
1.**准备工作**
到`SlAiCourse.Build.cs`引入Mode

![](https://i.imgur.com/a1iyOuE.png)

编译通过后再写相关代码，避免编译BUG

进入虚幻编辑器的项目设置新建两个碰撞类型`Enemy`,`EnemyTool`,设置为`Block`和`Ignore`
新建配置文件

![](https://i.imgur.com/g7LjrBl.png)

![](https://i.imgur.com/mFWZc4X.png)

然后设置其他配置文件
玩家和`EnemyTool`的碰撞设置为Overlay
玩家的工具和`Enemy`和`EnemyTool`的碰撞设置为Overlay
掉落物对`Enemy`的碰撞为Ignore

新建C++文件

![](https://i.imgur.com/Vy0xfD0.png)

![](https://i.imgur.com/2WEMycP.png)

![](https://i.imgur.com/zOSYWH2.png)

![](https://i.imgur.com/9dMVQGZ.png)

![](https://i.imgur.com/yGy31tb.png)

![](https://i.imgur.com/NIW57oN.png)

在蓝图文件夹下新建文件夹`Enemy`新建动作蓝图`Enemy_Animation`

![](https://i.imgur.com/ixMuB4Q.png)

添加动作

![](https://i.imgur.com/MK5QcpJ.png)

2.**完成敌人逻辑**
在`SlAiEnemyCharacter.cpp`引入头文件
```
#include "ConstructorHelpers.h"
#include "Components/CapsuleComponent.h"
#include "Components/SkeletalMeshComponent.h"
#include "Components/BoxComponent.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "Perception/PawnSensingComponent.h"
#include "SlAiEnemyController.h"
#include "SlAiEnemyTool.h"
```
在头文件声明`protected`类型：
```
	//武器插槽
	UPROPERTY(VisibleDefaultsOnly, Category = Mesh)
		class UChildActorComponent* WeaponSocket;
	//盾牌插槽
	UPROPERTY(VisibleDefaultsOnly, Category = Mesh)
		class UChildActorComponent* SheildSocket;
```
在构造函数下：
```
	//设置AI控制器
	AIControllerClass = ASlAiEnemyController::StaticClass();	
	//设置碰撞体属性文件
	GetCapsuleComponent()->SetCollisionProfileName(FName("EnemyProfile"));
	GetCapsuleComponent()->bGenerateOverlapEvents = true;
	//添加模型
	static ConstructorHelpers::FObjectFinder<USkeletalMesh> StaticEnemyMesh(TEXT("SkeletalMesh'/Game/Res/PolygonAdventure/Mannequin/Enemy/SkMesh/Enemy.Enemy'"));
	GetMesh()->SetSkeletalMesh(StaticEnemyMesh.Object);
	GetMesh()->SetCollisionObjectType(ECC_Pawn);
	GetMesh()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	GetMesh()->SetCollisionResponseToAllChannels(ECR_Ignore);
	GetMesh()->SetRelativeLocation(FVector(0.f, 0.f, -95.f));
	GetMesh()->SetRelativeRotation(FQuat::MakeFromEuler(FVector(0.f, 0.f, -90.f)));
	//添加动画蓝图
	static ConstructorHelpers::FClassFinder<UAnimInstance> StaticEnemyAnim(TEXT("AnimBlueprint'/Game/Blueprint/Enemy/Enemy_Animation.Enemy_Animation_C'"));
	GetMesh()->AnimClass = StaticEnemyAnim.Class;
	//实例化插槽
	WeaponSocket = CreateDefaultSubobject<UChildActorComponent>(TEXT("WeaponSocket"));
	SheildSocket = CreateDefaultSubobject<UChildActorComponent>(TEXT("SheildSocket"));
```
在`SlAiEnemyTool.h `下声明：
```
public:	
	//是否允许检测
	void ChangeOverlayDetect(bool IsOpen);
	static TSubclassOf<AActor> SpawnEnemyWeapon();
	static TSubclassOf<AActor> SpawnEnemySheild();
protected:
	UFUNCTION()
		virtual void OnOverlayBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult & SweepResult);
	UFUNCTION()
		virtual void OnOverlayEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
	//根组件
	USceneComponent* RootScene;
	//静态模型
	UPROPERTY(EditAnywhere, Category = Mesh)
		UStaticMeshComponent* BaseMesh;
	//盒子碰撞体
	UPROPERTY(EditAnywhere, Category = Mesh)
		class UBoxComponent* AffectCollision;
```
然后删除`BeginPlay`和`Tick`函数

添加头文件
```
#include "Components/StaticMeshComponent.h"
#include "Components/BoxComponent.h"
#include "Components/PrimitiveComponent.h"
#include "SlAiEnemyWeapon.h"
#include "SlAiEnemySheild.h"
```
在构造函数初始化：
```
	//实例化根节点
	RootScene = CreateDefaultSubobject<USceneComponent>("RootScene");
	RootComponent = RootScene;
	//在这里实现模型组件但是不进行模型绑定
	BaseMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BaseMesh"));
	BaseMesh->SetupAttachment(RootComponent);
	BaseMesh->SetCollisionProfileName(FName("NoCollision"));
	//实现碰撞组件
	AffectCollision = CreateDefaultSubobject<UBoxComponent>(TEXT("AffectCollision"));
	AffectCollision->SetupAttachment(BaseMesh);
	AffectCollision->SetCollisionProfileName(FName("EnemyToolProfile"));
	//初始时关闭Overlay检测
	AffectCollision->bGenerateOverlapEvents = false;
	//绑定检测方法到碰撞体
	FScriptDelegate OverlayBegin;
	OverlayBegin.BindUFunction(this, "OnOverlayBegin");
	AffectCollision->OnComponentBeginOverlap.Add(OverlayBegin);
	FScriptDelegate OverlayEnd;
	OverlayEnd.BindUFunction(this, "OnOverlayEnd");
	AffectCollision->OnComponentEndOverlap.Add(OverlayEnd);//绑定检测方法到碰撞体
```
两个Spawn函数和Change函数实现

![](https://i.imgur.com/lIUA4Jz.png)

在`SlAiEnemyWeapon.h `下声明构造函数
然后在构造函数里绑定模型
```
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Wep_GreatAxe_01.SM_Wep_GreatAxe_01'"));
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(-38.f, 9.6f, 9.8f));
	BaseMesh->SetRelativeRotation(FRotator(-10.f, 76.5f, -99.f));
	BaseMesh->SetRelativeScale3D(FVector(0.75f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 158.f));
	AffectCollision->SetRelativeScale3D(FVector(1.125f, 0.22f, 1.f));
```
引入头文件
```
#include "ConstructorHelpers.h"
#include "Components/StaticMeshComponent.h"
#include "Components/BoxComponent.h"
#include "SlAiPlayerCharacter.h"
```
在`SlAiEnemySheild.h`声明构造函数
在构造函数声明
```
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Wep_Sheild_01.SM_Wep_Sheild_01'"));
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(53.f, -3.f, -9.f));
	BaseMesh->SetRelativeRotation(FRotator(0.f, 90.f, 90.f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 43.f));
	AffectCollision->SetRelativeScale3D(FVector(0.8125f, 0.156f, 1.344f));
```
引入头文件
```
#include "ConstructorHelpers.h"
#include "Components/StaticMeshComponent.h"
#include "Components/BoxComponent.h"
```
在`SlAiEnemyCharacter.cpp`的`BeginPlay`下绑定插槽和物品
```
	//绑定插槽
	WeaponSocket->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetNotIncludingScale, FName("RHSocket"));
	SheildSocket->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetNotIncludingScale, FName("LHSocket"));
	//给插槽添加物品
	WeaponSocket->SetChildActorClass(ASlAiEnemyTool::SpawnEnemyWeapon());
	SheildSocket->SetChildActorClass(ASlAiEnemyTool::SpawnEnemySheild());
```
在`SlAiEnemyAnim.h `下声明构造函数和`public`的变量和函数
```
	virtual void NativeUpdateAnimation(float DeltaSeconds) override;
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = EnemyAnim)
		float Speed;
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = EnemyAnim)
		float IdleType;
protected:
	//保存角色
	class ASlAiEnemyCharacter* SECharacter;
```
在构造函数初始化参数
```
	//初始化参数
	Speed = 0.f;
	IdleType = 0.f;
```
引入头文件
```
#include "SlAiEnemyCharacter.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "Animation/AnimSequence.h"
#include "ConstructorHelpers.h"
```
在`NativeUpdateAnimation`中初始化速度和角色指针
```
	//初始化角色指针
	if (!SECharacter) SECharacter = Cast<ASlAiEnemyCharacter>(TryGetPawnOwner());
	if (!SECharacter) return;
	//设置速度
	Speed = SECharacter->GetVelocity().Size();
```
进入虚幻编辑器打开动作蓝图并删除添加的动作新建`State Machine`并重命名为`LocalMotion`
双击打开，新建新的`Static`并重命名为`Move`然后连接到根组件
双击打开`Move`
在资源栏中找到`Enemy Move`拖进蓝图然后连接参数

![](https://i.imgur.com/nmWG0HE.png)

将节点连好后，进入地图将所选中资源加入地图中

![](https://i.imgur.com/DEkEHAh.png)

运行游戏就可以看到左右手绑定的武器

##敌人血条和视野组件
1.**创建血条**
新建C++文件

![](https://i.imgur.com/AQTeVg2.png)

在`SlAiEnemyHPWidget`头文件内声明：
```
public:
	//提供给敌人角色来调用
	void ChangeHP(float HP);
private:
	TSharedPtr<class SProgressBar> HPBar;
	//结果颜色
	FLinearColor ResultColor;
```
然后再构造函数创建Widget
```
	ChildSlot
	[
		SAssignNew(HPBar, SProgressBar)
	];
```
改变血量：
```
	HP = FMath::Clamp<float>(HP, 0.f, 1.f);
	HPBar->SetPercent(HP);
	ResultColor = FLinearColor(1.f - HP, HP, 0.f, 1.f);
	HPBar->SetFillColorAndOpacity(FSlateColor(ResultColor));
```
在`SlAiEnemyCharacter`头文件声明：
```
public:
	//实时更新血条的朝向,由Controller调用,传入玩家位置
	void UpdateHPBarRotation(FVector SPLoaction);
protected:
	//血条
	UPROPERTY(EditAnywhere, Category = Mesh)
		class UWidgetComponent* HPBar;
	//敌人感知
	UPROPERTY(EditAnywhere, Category = Mesh)
		class UPawnSensingComponent* EnemySense;
private:
	//绑定到敌人感知的方法
	UFUNCTION()
		void OnSeePlayer(APawn* PlayerChar);
	//血条UI引用
	TSharedPtr<class SSlAiEnemyHPWidget> HPBarWidget;
	//控制器引用
	class ASlAiEnemyController* SEController;
	//血量
	float HP;
```

在`SlAiEnemyController`头文件下声明
```
public:
	virtual void Tick(float DeltaTime) override;
	//获取玩家的位置
	FVector GetPlayerLocation() const;
protected:
	virtual void BeginPlay() override;
private:
	//玩家的指针
	class ASlAiPlayerCharacter* SPCharacter;
	//敌人角色指针
	class ASlAiEnemyCharacter* SECharacter;
```
在构造函数中把`Tick`打开
引用头文件
```
#include "Kismet/GameplayStatics.h"
#include "SlAiPlayerCharacter.h"
#include "SlAiEnemyCharacter.h"
```
初步实现：

![](https://i.imgur.com/XG6dz21.png)

这时就可以回到`SlAiEnemyCharacter`的构造函数下实例化血条和敌人感知组件
```
	//实例化血条
	HPBar = CreateDefaultSubobject<UWidgetComponent>(TEXT("HPBar"));
	HPBar->AttachTo(RootComponent);
	//实例化敌人感知组件
	EnemySense = CreateDefaultSubobject<UPawnSensingComponent>(TEXT("EnemySense"));
```
并引入头文件
```
#include "WidgetComponent.h"
#include "SSlAiEnemyHPWidget.h"
```
在`BeginPlay`函数下设置血条和感知范围
```
	//设置血条widget
	SAssignNew(HPBarWidget, SSlAiEnemyHPWidget);
	HPBar->SetSlateWidget(HPBarWidget);
	HPBar->SetRelativeLocation(FVector(0.f, 0.f, 100.f));
	HPBar->SetDrawSize(FVector2D(100.f, 10.f));
	//设置初始血量
	HP = 200.f;
	HPBarWidget->ChangeHP(HP / 200.f);
	//敌人感知参数设置
	EnemySense->HearingThreshold = 0.f;
	EnemySense->LOSHearingThreshold = 0.f;
	EnemySense->SightRadius = 1000.f;
	EnemySense->SetPeripheralVisionAngle(55.f);
	EnemySense->bHearNoises = false;
	//绑定看到玩家的方法
	FScriptDelegate OnSeePlayerDele;
	OnSeePlayerDele.BindUFunction(this, "OnSeePlayer");
	EnemySense->OnSeePawn.Add(OnSeePlayerDele);
```
实现`UpdateHPBarRotation`
```
	FVector StartPos(GetActorLocation().X, GetActorLocation().Y, 0);
	FVector TargetPos(SPLoaction.X, SPLoaction.Y, 0.f);
	HPBar->SetWorldRotation(FRotationMatrix::MakeFromX(TargetPos - StartPos).Rotator());
```
实现`OnSeePlayer`

![](https://i.imgur.com/RCv894s.png)



此时就可以编译成功进入编辑器