#敌人攻击AI
##26.1 敌人攻击
1.**保存玩家距离与准备工作**
在`SlAiEnemyController`头文件下声明：
```
private:
	//保存与玩家的距离序列,保存8个,每半秒更新一次
	TArray<float> EPDisList;
	//时间委托句柄
	FTimerHandle EPDisHandle;
	//更新状态序列
	void UpdateStatePama();
public:
	//判定玩家是否在远离
	bool IsPlayerAway();
	//获取玩家指针
	UObject* GetPlayerPawn();
	//告诉控制器动作完成
	void ResetProcess(bool IsFinish);
```
在`BeginPlay`函数下进行绑定和计时器初始化
```
	//进行委托绑定
	FTimerDelegate EPDisDele = FTimerDelegate::CreateUObject(this, &ASlAiEnemyController::UpdateStatePama);
	GetWorld()->GetTimerManager().SetTimer(EPDisHandle, EPDisDele, 0.3f, true);
	//血量百分比初始化为1
	HPRatio = 1;
	//设置状态计时器
	IsAllowHurt = false;
	HurtTimeCount = 0.f;
```
实现`UpdateStatePama`函数
```
	//更新与玩家的距离序列
	if (EPDisList.Num() < 6)
	{
		EPDisList.Push(FVector::Distance(SECharacter->GetActorLocation(), GetPlayerLocation()));
	}
	else 
	{
		EPDisList.RemoveAt(0);
		EPDisList.Push(FVector::Distance(SECharacter->GetActorLocation(), GetPlayerLocation()));
	}
```
实现`IsPlayerAway`函数
```
	if (!IsLockPlayer || !SPCharacter || EPDisList.Num() < 6 || IsPlayerDead()) return false;
	int BiggerNum = 0;
	float LastDis = -1.f;
	//只要有三个比前面的大,就判定远离了
	for (TArray<float>::TIterator It(EPDisList); It; ++It)
	{
		if (*It > LastDis) BiggerNum += 1;
		LastDis = *It;
	}
	return BiggerNum > 3;
```
实现`GetPlayerPawn`
```
	return SPCharacter;
```
实现`ResetProcess`
```
	//修改完成状态
	BlackboardComp->SetValueAsBool("ProcessFinish", IsFinish);
```

在`SlAiEnemyCharacter`声明更新朝向：

![](https://i.imgur.com/6n9H9N8.png)
private：
![](https://i.imgur.com/dfv7CD3.png)

实现：

![](https://i.imgur.com/KLlzygZ.png)

![](https://i.imgur.com/2UJFlxj.png)


2.**设置敌人AI**

打开`SlAiEnemyTaskAttackSwitch`头文件声明：
```
	//重写执行函数
	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
protected:
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector AttackType;
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector EnemyState;
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector PlayerPawn;
```
实现之前先引入头文件
```
#include "SlAiEnemyController.h"
#include "SlAiEnemyCharacter.h"
#include "AI/Navigation/NavigationSystem.h"
#include "BehaviorTree/BehaviorTreeComponent.h"
#include "BehaviorTree/BlackboardComponent.h"
#include "SlAiTypes.h"
```
然后实现`ExecuteTask`函数
```
	//如果初始化敌人参数不成功,直接返回失败
	if (!InitEnemyElement(OwnerComp)) return EBTNodeResult::Failed;
	//给玩家指针赋值
	OwnerComp.GetBlackboardComponent()->SetValueAsObject(PlayerPawn.SelectedKeyName, SEController->GetPlayerPawn());
	//如果玩家已经死亡
	if (SEController->IsPlayerDead())
	{
		//告诉控制器丢失了玩家
		SEController->LoosePlayer();
		//修改状态为巡逻
		OwnerComp.GetBlackboardComponent()->SetValueAsEnum(EnemyState.SelectedKeyName, (uint8)EEnemyAIState::ES_Patrol);
		//跳出攻击状态
		return EBTNodeResult::Failed;
	}
	//获取与玩家的距离
	float EPDistance = FVector::Distance(SECharacter->GetActorLocation(), SEController->GetPlayerLocation());
	//如果距离小于200
	if (EPDistance < 200.f)
	{
		//修改攻击状态为普攻
		OwnerComp.GetBlackboardComponent()->SetValueAsEnum(AttackType.SelectedKeyName, (uint8)EEnemyAttackType::EA_Normal);
		return EBTNodeResult::Succeeded;
	}
	//如果距离小于300并且判定到玩家在远离
	if (EPDistance < 300.f && SEController->IsPlayerAway())
	{
		//修改状态为追逐攻击
		OwnerComp.GetBlackboardComponent()->SetValueAsEnum(AttackType.SelectedKeyName, (uint8)EEnemyAttackType::EA_Pursuit);
		return EBTNodeResult::Succeeded;
	}
	if (EPDistance > 200.f && EPDistance < 300.f) 
	{
		//修改攻击状态为冲刺
		OwnerComp.GetBlackboardComponent()->SetValueAsEnum(AttackType.SelectedKeyName, (uint8)EEnemyAttackType::EA_Dash);
		return EBTNodeResult::Succeeded;
	}
	//如果大于300
	if (EPDistance > 300.f)
	{
		//修改攻击状态为追逐
		OwnerComp.GetBlackboardComponent()->SetValueAsEnum(EnemyState.SelectedKeyName, (uint8)EEnemyAIState::ES_Chase);
		//跳出攻击状态
		return EBTNodeResult::Failed;
	}
	return EBTNodeResult::Failed;
```
打开`SlAiEnemyTaskAttackNormal`头文件声明：
```
		//重写执行函数
		virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
protected:
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector WaitTime;
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector PlayerPawn;
```
并引入相同的头文件
实现`ExecuteTask`函数
```
	//如果初始化敌人参数不成功,直接返回失败
	if (!InitEnemyElement(OwnerComp)) return EBTNodeResult::Failed;
	//播放普通攻击动画
	float AttackDuration = SECharacter->PlayAttackAction(EEnemyAttackType::EA_Normal);
	//设置参数
	OwnerComp.GetBlackboardComponent()->SetValueAsFloat(WaitTime.SelectedKeyName, AttackDuration);
	//返回成功
	return EBTNodeResult::Succeeded;
```

在`SlAiEnemyTaskAttackDash`头文件声明：
```
	//重写执行函数
	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
	//重写任务终止函数
	virtual EBTNodeResult::Type AbortTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;	
protected:
	//动作结束后事件
	void OnAnimationTimerDone();
protected:
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector WaitTime;
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector Destination;
	//攻击动作结束后的委托
	FTimerHandle TimerHandle;
```
同样引入相同的头文件
并加上
```
#include "TimerManager.h"
```
实现`ExecuteTask`函数
```
	//如果初始化敌人参数不成功,直接返回失败
	if (!InitEnemyElement(OwnerComp)) return EBTNodeResult::Failed;
	//播放突击动画
	float AttackDuration = SECharacter->PlayAttackAction(EEnemyAttackType::EA_Dash);
	//范围是0
	const float ChaseRadius = 5.f;
	//获取玩家到敌人之间的单位向量
	FVector SPToSE = SEController->GetPlayerLocation() - SECharacter->GetActorLocation();
	SPToSE.Normalize();
	//探索起点是玩家位置减去与敌人之间距离的一点点
	const FVector ChaseOrigin = SEController->GetPlayerLocation() - 20.f * SPToSE;
	//保存随机的位置
	FVector DesLoc(0.f);
	//使用导航系统获取随机点
	UNavigationSystem::K2_GetRandomReachablePointInRadius(SEController, ChaseOrigin, DesLoc, ChaseRadius);
	//角色速度
	float Speed = (FVector::Distance(SECharacter->GetActorLocation(), DesLoc)) / AttackDuration + 30.f;
	//修改敌人速度
	SECharacter->SetMaxSpeed(Speed);
	//修改目的地
	OwnerComp.GetBlackboardComponent()->SetValueAsVector(Destination.SelectedKeyName, DesLoc);
	//设置参数
	OwnerComp.GetBlackboardComponent()->SetValueAsFloat(WaitTime.SelectedKeyName, AttackDuration);
	//添加事件委托
	FTimerDelegate TimerDelegate = FTimerDelegate::CreateUObject(this, &USlAiEnemyTaskAttackDash::OnAnimationTimerDone);
	//注册到事件管理器
	SEController->GetWorld()->GetTimerManager().SetTimer(TimerHandle, TimerDelegate, AttackDuration, false);
	//继续执行
	return EBTNodeResult::Succeeded;
```
实现`AbortTask`函数
```
	//如果初始化敌人参数不成功或者事件句柄没有激活,直接返回
	if (!InitEnemyElement(OwnerComp) || !TimerHandle.IsValid()) return EBTNodeResult::Aborted;
	//卸载时间委托
	SEController->GetWorld()->GetTimerManager().ClearTimer(TimerHandle);
	//返回
	return EBTNodeResult::Aborted;
```
实现`OnAnimationTimerDone`函数
```
	//重新设置速度为300
	if (SECharacter) SECharacter->SetMaxSpeed(300.f);
```

在`SlAiEnemyTaskAttackPursuit`头文件声明：
```
	//重写执行函数
	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
	//重写任务终止函数
	virtual EBTNodeResult::Type AbortTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
protected:
	//动作结束后事件
	void OnAnimationTimerDone();
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector IsActionFinish;
	//攻击动作结束后的委托
	FTimerHandle TimerHandle;
```
在CPP文件引入头文件
```
#include "SlAiEnemyController.h"
#include "SlAiEnemyCharacter.h"
#include "AI/Navigation/NavigationSystem.h"
#include "BehaviorTree/BehaviorTreeComponent.h"
#include "BehaviorTree/BlackboardComponent.h"
#include "TimerManager.h"
```
`ExecuteTask`函数实现
```
	//如果初始化敌人参数不成功,直接返回失败
	if (!InitEnemyElement(OwnerComp)) return EBTNodeResult::Failed;
	//播放追击动画
	float AttackDuration = SECharacter->PlayAttackAction(EEnemyAttackType::EA_Pursuit);
	//设置速度为600,不小于玩家
	SECharacter->SetMaxSpeed(600.f);
	//设置参数
	OwnerComp.GetBlackboardComponent()->SetValueAsBool(IsActionFinish.SelectedKeyName, false);
	//添加事件委托
	FTimerDelegate TimerDelegate = FTimerDelegate::CreateUObject(this, &USlAiEnemyTaskAttackPursuit::OnAnimationTimerDone);
	//注册到事件管理器
	SEController->GetWorld()->GetTimerManager().SetTimer(TimerHandle, TimerDelegate, AttackDuration, false);
	return EBTNodeResult::Succeeded;
```
`AbortTask`函数实现
```
	//如果初始化敌人参数不成功或者事件句柄没有激活,直接返回
	if (!InitEnemyElement(OwnerComp) || !TimerHandle.IsValid()) return EBTNodeResult::Aborted;
	//卸载时间委托
	SEController->GetWorld()->GetTimerManager().ClearTimer(TimerHandle);
	//返回
	return EBTNodeResult::Aborted;
```
`OnAnimationTimerDone`函数实现：
```
	//设置动作完成
	if (SEController) SEController->ResetProcess(true);
	//修改速度回300
	if (SECharacter) SECharacter->SetMaxSpeed(300.f);
```

在`SlAiEnemyTaskAttackFollow`头文件声明：
```
		//重写执行函数
		virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
protected:
	UPROPERTY(EditAnywhere, Category = "Blackboard")
		struct FBlackboardKeySelector Destination;
```
在CPP文件引入头文件
```
#include "SlAiEnemyController.h"
#include "SlAiEnemyCharacter.h"
#include "AI/Navigation/NavigationSystem.h"
#include "BehaviorTree/BehaviorTreeComponent.h"
#include "BehaviorTree/BlackboardComponent.h"
```
实现`ExecuteTask`函数
```
	//如果初始化敌人参数不成功,直接返回失败
	if (!InitEnemyElement(OwnerComp)) return EBTNodeResult::Failed;
	//范围是5
	const float ChaseRadius = 0.f;
	//获取玩家到敌人之间的单位向量
	FVector SPToSE = SEController->GetPlayerLocation() - SECharacter->GetActorLocation();
	//获取距离
	float EPDistance = SPToSE.Size();
	//如果距离大于100.f,获取玩家与敌人连线上距离玩家100.f的那个点作为原始点来寻找导航点
	if (EPDistance > 100.f) {
		//归一化
		SPToSE.Normalize();
		//探索起点是玩家位置减去与敌人之间距离的一点点
		const FVector ChaseOrigin = SEController->GetPlayerLocation() - 100.f * SPToSE;
		//保存随机的位置
		FVector DesLoc(0.f);
		//使用导航系统获取随机点
		UNavigationSystem::K2_GetRandomReachablePointInRadius(SEController, ChaseOrigin, DesLoc, ChaseRadius);
		//修改目的地
		OwnerComp.GetBlackboardComponent()->SetValueAsVector(Destination.SelectedKeyName, DesLoc);
	}
	else
	{
		//如果距离小于100.f,那么设置敌人当前的位置为目标位置
		OwnerComp.GetBlackboardComponent()->SetValueAsVector(Destination.SelectedKeyName, SECharacter->GetActorLocation());
	}
	//返回成功
	return EBTNodeResult::Succeeded;
```

在`SlAiEnemyTaskRotate`头文件声明：
```
		//重写执行函数
		virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
```
引入相同的头文件
实现`ExecuteTask`函数

![](https://i.imgur.com/lYhWj1P.png)

编译通过后进入编辑器设置行为树蓝图

![](https://i.imgur.com/85wRyJD.png)

![](https://i.imgur.com/TkG1fBJ.png)

![](https://i.imgur.com/KqvcOQC.png)

![](https://i.imgur.com/c4nqVV7.png)

![](https://i.imgur.com/TJYpmoZ.png)

![](https://i.imgur.com/vblxuQ8.png)

![](https://i.imgur.com/Z1rnwyg.png)

![](https://i.imgur.com/y7TxDS3.png)

![](https://i.imgur.com/8FED6k6.png)

![](https://i.imgur.com/gN1ILtw.png)

![](https://i.imgur.com/C9xzaYj.png)

![](https://i.imgur.com/p8w8BtM.png)

![](https://i.imgur.com/WsHwPeb.png)

![](https://i.imgur.com/TAQAhSm.png)

![](https://i.imgur.com/lBmGINc.png)

![](https://i.imgur.com/HK0duIu.png)
