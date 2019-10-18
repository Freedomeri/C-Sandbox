#生成掉落物与动态创建材质和检测
##17.1生成掉落物与动态创建材质
1.**设置捡东西时不显示手上物品**
在`SlAiPlayerAnim.h`添加：
```
	//开启和关闭手上物品的显示与否,在捡东西的时候调用
	UFUNCTION(BlueprintCallable, Category = PlayAnim)
		void RenderHandObject(bool IsRender);
```
实现：
```
	if (!SPCharacter) return;
	SPCharacter->RenderHandObject(IsRender);
```
在`ASlAiPlayerCharacter.h `添加`public`函数
```
	//是否渲染手上物品,由Anim进行调用
	void RenderHandObject(bool IsRender);
```
实现：
```
	//如果手上物品木有
	if (!HandObject->GetChildActor()) return;
	//如果有物品
	HandObject->GetChildActor()->SetActorHiddenInGame(!IsRender);
```
编译通过后进入虚幻编辑器
在Montage动作蓝图绑定写好的`Renderhandobject`

![](https://i.imgur.com/69GLJj4.png)

2.**设置掉落物的碰撞类型**
在项目设置里新建`Flob`的碰撞类型为`Block`
配置文件为：

![](https://i.imgur.com/ylhljNB.png)

将`PlayerProfile`与`Flob`设置为overlap
将`ToolProfile`与`Flob`设置为ignore

3.**掉落物的生成**
新建C++文件

![](https://i.imgur.com/oZAuOwi.png)

新建材质

![](https://i.imgur.com/BCbWLTt.png)

材质蓝图：

![](https://i.imgur.com/HmeerSA.png)

设置材质两面都显示

![](https://i.imgur.com/bCqiHIF.png)


然后右键材质新建材质实例：

![](https://i.imgur.com/f33qAg0.png)

![](https://i.imgur.com/eAmJZA0.png)

在`SlAiFlobObject.h `创建组件
```
private:
	class UBoxComponent* BoxCollision;
	class UStaticMeshComponent* BaseMesh;
	//物品ID
	int ObjectIndex;
	//渲染贴图
	void RenderTexture();
public:
	//生成物品初始化
	void CreateFlobObject(int ObjectID);
	class UTexture* ObjectIconTex;
	class UMaterialInstanceDynamic* ObjectIconMatDynamic;
```
并引入数据结构的头文件

在构造函数初始化
```
	BoxCollision = CreateDefaultSubobject<UBoxComponent>(TEXT("BoxCollision"));
	RootComponent = (USceneComponent*)BoxCollision;
	//设置碰撞属性
	BoxCollision->SetCollisionProfileName(FName("FlobProfile"));
	//启动物体模拟
	BoxCollision->SetSimulatePhysics(true);
	//锁定旋转
	BoxCollision->SetConstraintMode(EDOFMode::Default);
	BoxCollision->GetBodyInstance()->bLockXRotation = true;
	BoxCollision->GetBodyInstance()->bLockYRotation = true;
	BoxCollision->GetBodyInstance()->bLockZRotation = true;
	//设置大小
	BoxCollision->SetBoxExtent(FVector(15.f));
	BaseMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BaseMesh"));
	BaseMesh->SetupAttachment(RootComponent);
	static ConstructorHelpers::FObjectFinder<UStaticMesh> StaticBaseMesh(TEXT("StaticMesh'/Engine/BasicShapes/Plane.Plane'"));
	BaseMesh->SetStaticMesh(StaticBaseMesh.Object);
	BaseMesh->SetCollisionResponseToChannels(ECR_Ignore);
	BaseMesh->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	//设置变换
	BaseMesh->SetRelativeRotation(FRotator(0.f, 0.f, 90.f));
	BaseMesh->SetRelativeScale3D(FVector(0.3f));
	UMaterialInterface* StaticObjectIconMat = LoadObject<UMaterialInterface>(NULL, TEXT("MaterialInstanceConstant'/Game/Material/FlobIconMatInst.FlobIconMatInst'"));
	//动态创建材质
	ObjectIconMatDynamic = UMaterialInstanceDynamic::Create(StaticObjectIconMat, nullptr);
```
引入头文件
```
#include "ConstructorHelpers.h"
#include "Components/StaticMeshComponent.h"
#include "Components/BoxComponent.h"
#include "Materials/MaterialInstanceDynamic.h"
```
在`RenderTexture`实现：
```
	TSharedPtr<ObjectAttribute> ObjectAttr = *SlAiDataHandle::Get()->ObjectAttrMap.Find(ObjectIndex);
	ObjectIconTex = LoadObject<UTexture>(NULL, *ObjectAttr->TexPath);
	ObjectIconMatDynamic->SetTextureParameterValue(FName("ObjectTex"), ObjectIconTex);
	BaseMesh->SetMaterial(0, ObjectIconMatDynamic);
```

在`CreateFlobObject`函数

![](https://i.imgur.com/VHgDwOE.png)

在`Tick`函数添加：
```
	Super::Tick(DeltaTime);
	//一直旋转
	BaseMesh->AddLocalRotation(FRotator(DeltaTime * 60.f, 0.f, 0.f));
```
在`SlAiResourceObject.h`声明变量
```
protected:
	//生成掉落物
	void CreateFlobObject();
```
在`.cpp`文件引入头文件
```
#include "SlAiFlobObject.h"
```
实现：
```
TSharedPtr<ResourceAttribute> ResourceAttr = *SlAiDataHandle::Get()->ResourceAttrMap.Find(ResourceIndex);
	//遍历生成掉落物
	for (TArray<TArray<int>>::TIterator It(ResourceAttr->FlobObjectInfo); It; ++It) {
		//随机生成的数量
		FRandomStream Stream;
		Stream.GenerateNewSeed();
		//生成数量
		int Num = Stream.RandRange((*It)[1], (*It)[2]);

		if (GetWorld()) {
			for (int i = 0; i < Num; ++i) {
				//生成掉落物
				ASlAiFlobObject* FlobObject = GetWorld()->SpawnActor<ASlAiFlobObject>(GetActorLocation() + FVector(0.f, 0.f, 20.f), FRotator::ZeroRotator);
				FlobObject->CreateFlobObject((*It)[0]);
			}
		}
	}
```
编译通过就可以破坏东西后掉落物品

##17.2 动态检测
在`SlAiFlobObject.h `声明`private`变量和函数
```
	//玩家指针
	class ASlAiPlayerCharacter* SPCharacter;
	//动态检测Timer
	FTimerHandle DetectTimer;
	//销毁Timer
	FTimerHandle DestroyTimer;
	//动态检测事件
	void DetectPlayer();
	//销毁事件
	void DestroyEvent();
```
在`BeginPlay`函数内实现：
```
	//检测世界是否存在
	if (!GetWorld()) return;
	//注册检测事件
	FTimerDelegate DetectPlayerDele;
	DetectPlayerDele.BindUObject(this, &ASlAiFlobObject::DetectPlayer);
	//每秒运行一次,循环运行,延迟3秒运行
	GetWorld()->GetTimerManager().SetTimer(DetectTimer, DetectPlayerDele, 1.f, true, 3.f);
	//注册销毁事件
	FTimerDelegate DestroyDele;
	DestroyDele.BindUObject(this, &ASlAiFlobObject::DestroyEvent);
	GetWorld()->GetTimerManager().SetTimer(DestroyTimer, DestroyDele, 30.f, false);
	//初始玩家指针为空
	SPCharacter = NULL;
```
在`DetectPlayer`函数实现：
```
	//检测世界是否存在
	if (!GetWorld()) return;
	//保存检测结果
	TArray<FOverlapResult> Overlaps;
	FCollisionObjectQueryParams ObjectParams;
	FCollisionQueryParams Params;
	Params.AddIgnoredActor(this);
	Params.bTraceAsyncScene = true;
	//进行动态检测,检测范围是200,检测成功的话返回true
	if (GetWorld()->OverlapMultiByObjectType(Overlaps, GetActorLocation(), FQuat::Identity, ObjectParams, FCollisionShape::MakeSphere(200.f), Params))
	{
		for (TArray<FOverlapResult>::TIterator It(Overlaps); It; ++It) {
			//如果检测到了玩家
			if (Cast<ASlAiPlayerCharacter>(It->GetActor())) {
				//赋值
				SPCharacter = Cast<ASlAiPlayerCharacter>(It->GetActor());
				//如果背包有空间,后面再添加函数
				if (SPCharacter->IsPackageFree(ObjectIndex))
				{
					//停止检测
					GetWorld()->GetTimerManager().PauseTimer(DetectTimer);
					//停止销毁定时器
					GetWorld()->GetTimerManager().PauseTimer(DestroyTimer);
					//关闭物理模拟
					BoxCollision->SetSimulatePhysics(false);
				}
				return;
			}
		}
	}
```
在`Tick`函数内加入动态检测：
```
	//如果检测到玩家
	if (SPCharacter) {
		//靠近玩家
		SetActorLocation(FMath::VInterpTo(GetActorLocation(), SPCharacter->GetActorLocation() + FVector(0.f, 0.f, 40.f), DeltaTime, 5.f));
		//如果距离接近0
		if (FVector::Distance(GetActorLocation(), SPCharacter->GetActorLocation() + FVector(0.f, 0.f, 40.f)) < 10.f)
		{
			//判断玩家背包是否有空间
			if (SPCharacter->IsPackageFree(ObjectIndex)) {
				//添加对应的物品到背包
				SPCharacter->AddPackageObject(ObjectIndex);
				//销毁自己
				DestroyEvent();
			}
			else {
				//如果玩家背包不为空,重置参数
				SPCharacter = NULL;
				//唤醒检测
				GetWorld()->GetTimerManager().UnPauseTimer(DetectTimer);
				//唤醒销毁线程
				GetWorld()->GetTimerManager().UnPauseTimer(DestroyTimer);
				//开启物理模拟
				BoxCollision->SetSimulatePhysics(true);
			}
		}
	}
```
在`DestroyEvent`加入销毁自身和定时器的功能：
```
	if (!GetWorld()) return;
	//注销定时器
	GetWorld()->GetTimerManager().ClearTimer(DetectTimer);
	GetWorld()->GetTimerManager().ClearTimer(DestroyTimer);
	//销毁自己
	GetWorld()->DestroyActor(this);
```
编译通过后就可以把地上的物品拾取到背包