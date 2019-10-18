#物体碰撞与动作切换
##14.1 物品类与碰撞事件绑定
1.**物体碰撞设置**
打开虚幻4编辑器，进入项目设置->Collision->Object Channels添加新的碰撞类型`Tool`设置为Overlap
在`preset`新建新的配置文件`ToolProfile`

![](https://i.imgur.com/n3XhAU9.png)

打开`PlayerProfile`修改对`Tool`的碰撞为`Ignore`

进入`SlAiPlayerCharacter.cpp`注释掉一行代码

![](https://i.imgur.com/YybdhAQ.png)

打开`SlAiHandleObject.h`创建变量和函数
```
protected：
//根组件
	class USceneComponent* RootScene;
	//盒子碰撞体
	UPROPERTY(EditAnywhere, Category = "SlAi")
		class UBoxComponent* AffectCollision;
	UFUNCTION()
		virtual void OnOverlayBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult & SweepResult);
	UFUNCTION()
		virtual void OnOverlayEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
public：
//物品ID
	int ObjectIndex;

```

用于对静态模型的控制和碰撞检测

将构造函数中代码替换为：
```
	PrimaryActorTick.bCanEverTick = true;
	//实例化根组件
	RootScene = CreateDefaultSubobject<USceneComponent>(TEXT("RootScene"));
	RootComponent = RootScene;
	//创建静态模型组件
	BaseMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BaseMesh"));
	BaseMesh->SetupAttachment(RootComponent);
	BaseMesh->SetCollisionProfileName(FName("NoCollision"));
	//实现碰撞组件
	AffectCollision = CreateDefaultSubobject<UBoxComponent>(TEXT("AffectCollision"));
	AffectCollision->SetupAttachment(RootComponent);
	AffectCollision->SetCollisionProfileName(FName("ToolProfile"));
	//初始时关闭Overlay检测
	AffectCollision->bGenerateOverlapEvents = false;
	//绑定检测方法到碰撞体
	FScriptDelegate OverlayBegin;
	OverlayBegin.BindUFunction(this, "OnOverlayBegin");
	AffectCollision->OnComponentBeginOverlap.Add(OverlayBegin);
	FScriptDelegate OverlayEnd;
	OverlayEnd.BindUFunction(this, "OnOverlayEnd");
	AffectCollision->OnComponentEndOverlap.Add(OverlayEnd);
```

进入`SlAiHandAxe.h`创建构造函数和`BeginPlay`

![](https://i.imgur.com/ebghGaw.png)

构造函数添加
```
	//给模型组件添加模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Wep_Axe_01.SM_Wep_Axe_01'"));
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(-28.f, -4.9f, 3.23f));
	BaseMesh->SetRelativeRotation(FRotator(0.f, -90.f, 90.f));
	BaseMesh->SetRelativeScale3D(FVector(1.f, 1.f, 1.f));
	//设置碰撞体属性
	AffectCollision->SetRelativeLocation(FVector(32.f, -5.f, 3.f));
	AffectCollision->SetRelativeScale3D(FVector(0.375f, 0.5f, 0.125f));
```
引入头文件
```
#include "Components/StaticMeshComponent.h"
#include "ConstructorHelpers.h"
#include "Components/BoxComponent.h"
```
在`BeginPlay`添加
```
Super::BeginPlay();
	//定义物品序号
	ObjectIndex = 5;
```
之后就要为所有物品依次添加模型和序号

因为没有单独的锤子资源所以在`ASlAiHandHammer.h`单独添加
```
public:
	ASlAiHandHammer();
protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;
protected:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Mesh)
		UStaticMeshComponent* ExtendMesh;
```
构造函数中添加：
```
//给模型组件添加上模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Bld_FencePost_01.SM_Bld_FencePost_01'"));
	//绑定模型到Mesh组件
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	//添加扩展模型组件
	ExtendMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ExtendMesh"));
	ExtendMesh->SetupAttachment(RootComponent);
	ExtendMesh->SetCollisionProfileName(FName("NoCollision"));
	//绑定模型到扩展模型组件
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticExtendMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Prop_StoneBlock_01.SM_Prop_StoneBlock_01'"));
	//绑定模型到Mesh组件
	ExtendMesh->SetStaticMesh(StaticExtendMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(35.f, 0.f, 3.f));
	BaseMesh->SetRelativeRotation(FRotator(0.f, -90.f, -90.f));
	BaseMesh->SetRelativeScale3D(FVector(0.4f, 0.4f, 0.4f));
	ExtendMesh->SetRelativeLocation(FVector(33.f, 1.f, 3.f));
	ExtendMesh->SetRelativeRotation(FRotator(0.f, -90.f, -90.f));
	ExtendMesh->SetRelativeScale3D(FVector(0.4f, 0.4f, 0.4f));
	//设置碰撞体属性
	AffectCollision->SetRelativeLocation(FVector(26.f, 1.f, 3.f));
	AffectCollision->SetRelativeScale3D(FVector(0.22f, 0.44f, 0.31f));
```
物品序号是6，在`BeginPlay`设置
```
Super::BeginPlay();
	//定义物品序号
	ObjectIndex = 6;
```

`SlAiHandApple`的代码：
```
	//给模型组件添加上模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Item_Fruit_02.SM_Item_Fruit_02'"));
	//绑定模型到Mesh组件
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(-8.f, -3.f, 7.f));
	BaseMesh->SetRelativeRotation(FRotator(-90.f, 0.f, 0.f));
	BaseMesh->SetRelativeScale3D(FVector(1.f, 1.f, 1.f));
	//设置碰撞盒属性
	AffectCollision->SetBoxExtent(FVector(10.f, 10.f, 10.f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 10.f));
```
序号是3
`SlAiHandMeat`的代码：
```
	//给模型组件添加上模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Prop_Meat_02.SM_Prop_Meat_02'"));
	//绑定模型到Mesh组件
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(6.f, -7.044f, 2.62f));
	BaseMesh->SetRelativeRotation(FRotator(-50.f, 90.f, 0.f));
	BaseMesh->SetRelativeScale3D(FVector(0.75f, 0.75f, 0.75f));
	//设置碰撞盒属性
	AffectCollision->SetBoxExtent(FVector(10.f, 10.f, 10.f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 10.f));
```
序号是4

`SlAiHandNone`的代码
```
	//不用绑定模型
	//设置碰撞盒属性
	AffectCollision->SetBoxExtent(FVector(10.f, 10.f, 10.f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 10.f));
```
序号是0

`SlAiHandStone`的代码
```
//给模型组件添加上模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Prop_StoneBlock_01.SM_Prop_StoneBlock_01'"));
	//绑定模型到Mesh组件
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(1.f, -1.414f, 7.071f));
	BaseMesh->SetRelativeRotation(FRotator(0.f, 0.f, -135.f));
	BaseMesh->SetRelativeScale3D(FVector(0.25f, 0.25f, 0.25f));
	//设置碰撞盒属性
	AffectCollision->SetBoxExtent(FVector(10.f, 10.f, 10.f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 10.f));
```
序号是2

`SlAiHandSword`的代码
```
//给模型组件添加上模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Wep_Sword_01.SM_Wep_Sword_01'"));
	//绑定模型到Mesh组件
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeLocation(FVector(-15.f, 1.f, 2.f));
	BaseMesh->SetRelativeRotation(FRotator(-20.f, 90.f, -90.f));
	BaseMesh->SetRelativeScale3D(FVector(0.8f, 0.8f, 1.f));
	//设置碰撞体属性
	AffectCollision->SetRelativeLocation(FVector(62.f, 1.f, 2.f));
	AffectCollision->SetRelativeRotation(FRotator(0.f, 0.f, 20.f));
	AffectCollision->SetRelativeScale3D(FVector(1.5f, 0.19f, 0.1f));
```
`SlAiHandWood`的代码
```
	//给模型组件添加上模型
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Game/Res/PolygonAdventure/Meshes/SM_Env_TreeLog_01.SM_Env_TreeLog_01'"));
	//绑定模型到Mesh组件
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetRelativeScale3D(FVector(0.1f, 0.1f, 0.1f));
	BaseMesh->SetRelativeRotation(FRotator(0.f, -20.f, 0.f));
	AffectCollision->SetBoxExtent(FVector(10.f, 10.f, 10.f));
	AffectCollision->SetRelativeLocation(FVector(0.f, 0.f, 10.f));
```
序号是1

通过编译后
可以通过写好的基类代码创建蓝图

![](https://i.imgur.com/i3a0Kap.png)

##14.2 手持物品与动作切换
1.**根据物品序号返回物品的方法**

在`ASlAiHandObject.h`声明`public`函数
```
	//根据物品ID返回物品的工厂方法
	static TSubclassOf<AActor> SpawnHandObject(int ObjectID);
```
实现：
```
	switch (ObjectID)
	{
	case 0:
		return ASlAiHandNone::StaticClass();
	case 1:
		return ASlAiHandWood::StaticClass();
	case 2:
		return ASlAiHandStone::StaticClass();
	case 3:
		return ASlAiHandApple::StaticClass();
	case 4:
		return ASlAiHandMeat::StaticClass();
	case 5:
		return ASlAiHandAxe::StaticClass();
	case 6:
		return ASlAiHandHammer::StaticClass();
	case 7:
		return ASlAiHandSword::StaticClass();
	}
	return ASlAiHandNone::StaticClass();
```
并引入头文件
```
#include "SlAiHandNone.h"
#include "SlAiHandWood.h"
#include "SlAiHandStone.h"
#include "SlAiHandApple.h"
#include "SlAiHandMeat.h"
#include "SlAiHandAxe.h"
#include "SlAiHandHammer.h"
#include "SlAiHandSword.h"
```

打开`SlAiPlayerCharacter.cpp`将`BeginPlay`注释掉的代码替换为：
```
	//添加Actor到HandObject
	HandObject->SetChildActorClass(ASlAiHandObject::SpawnHandObject(0));
```
2.**设置根据滑轮滚动选中物品并返回的方法**
在`SlAiPlayerCharacter.h`声明`public`函数
```
	//修改当前的手持物品
	void ChangeHandObject(TSubclassOf<AActor> HandObjectClass);
```
实现：

![](https://i.imgur.com/xmhRlDy.png)

在`SlAiPlayerController.h`声明相同名字的函数：

![](https://i.imgur.com/IhQ7XBe.png)

实现：
```
	//生成手持物品
	SPCharacter->ChangeHandObject(ASlAiHandObject::SpawnHandObject(SPState->GetCurrentHandObjectIndex()));
```
并引入头文件
```
#include "SlAiHandObject.h"
```
在`SlAiPlayerCharacter.cpp`修改`Changeview`在每个视角后面添加修改物品绑定位置的代码

![](https://i.imgur.com/WTsp1N4.png)

在`SlAiPlayerState.h `声明
```
    //获取当前手持物品的物品类型
	EObjectType::Type GetCurrentObjectType();
```
实现：
```
	TSharedPtr<ObjectAttribute> ObjectAttr;
	ObjectAttr = *SlAiDataHandle::Get()->ObjectAttrMap.Find(GetCurrentHandObjectIndex());
	return ObjectAttr->ObjectType;
```

在`ASlAiPlayerController.h`声明:
```
	//修改预动作
	void ChangePreUpperType(EUpperBody::Type RightType);
```
实现：
```
//根据当前手持物品的类型来修改预动作
	switch (SPState->GetCurrentObjectType())
	{
	case EObjectType::Normal:
		LeftUpperType = EUpperBody::Punch;
		RightUpperType = RightType;
		break;
	case EObjectType::Food:
		LeftUpperType = EUpperBody::Punch;
		//如果右键状态是拾取那就给拾取,拾取优先级高
		RightUpperType = RightType == EUpperBody::None ? EUpperBody::Eat : RightType;
		break;
	case EObjectType::Tool:
		LeftUpperType = EUpperBody::Hit;
		RightUpperType = RightType;
		break;
	case EObjectType::Weapon:
		LeftUpperType = EUpperBody::Fight;
		RightUpperType = RightType;
		break;
	}
```
在`Tick`函数加入临时代码替代射线检测

![](https://i.imgur.com/6GSV12Q.png)

在将`SlAiHandObject.cpp`中碰撞检测临时打开

![](https://i.imgur.com/hGDALBx.png)

引入Debug检测碰撞是否触发

![](https://i.imgur.com/DfblBSI.png)

打开场景的测试物体（随意）打开碰撞检测

![](https://i.imgur.com/m7UzdKQ.png)
*此时只有打开的物体才可以进行交互*

编译通过就可以看到不同动作对应物体的切换和碰撞检测
