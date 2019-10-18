#读取存档和保存
##32.1 读取存档
1.**先在地图上放置足够多的资源和敌人**

2.**新建C++文件**

![](https://i.imgur.com/wS54Vxk.png)

3.**实现功能**
在`SlAiPackageManager`头文件声明public函数
```
	//加载存档
	void LoadRecord(TArray<int32>* InputIndex, TArray<int32>* InputNum, TArray<int32>* NormalIndex, TArray<int32>* NormalNum, TArray<int32>* ShortcutIndex, TArray<int32>* ShortcutNum);
```
实现：
```
	for (int i = 0; i < InputContainerList.Num(); ++i)
	{
		if ((*InputIndex)[i] != 0) InputContainerList[i]->ResetContainerPara((*InputIndex)[i], (*InputNum)[i]);
	}
	for (int i = 0; i < NormalContainerList.Num(); ++i) {
		if ((*NormalIndex)[i] != 0) NormalContainerList[i]->ResetContainerPara((*NormalIndex)[i], (*NormalNum)[i]);
	}
	for (int i = 0; i < ShortcutContainerList.Num(); ++i) {
		if ((*ShortcutIndex)[i] != 0) ShortcutContainerList[i]->ResetContainerPara((*ShortcutIndex)[i], (*ShortcutNum)[i]);
	}
```
在`SlAiPlayerState.h`声明public函数
```
void LoadState(float HPVal, float HungerVal);
```
实现：
```
	HP = HPVal;
	Hunger = HungerVal;
	//执行修改玩家状态UI的委托
	UpdateStateWidget.ExecuteIfBound(HP / 500.f, Hunger / 500.f);
```
在`SLAiEnemyCharacter.h`声明public函数
```
void LoadHP(float HPVal);
bool IsDestoryNextTick;
```
实现加载血量：
```
	HP = HPVal;
	//修改血量显示
	HPBarWidget->ChangeHP(HP / 200.f);
```
在构造函数初始化：
```
	//设置下一帧不销毁自己,得放在构造函数进行初始化,避免与GameMode的加载函数冲突
	IsDestroyNextTick = false;
```
再在Tick函数进行判断
```
	//如果准备销毁为true,进行销毁
	if (IsDestroyNextTick) DestroyEvent();
```

在`SlAiSaveGame`声明：
```
public:

	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		FVector PlayerLocation;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		float PlayerHP;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		float PlayerHunger;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<int32> InputIndex;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<int32> InputNum;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<int32> NormalIndex;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<int32> NormalNum;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<int32> ShortcutIndex;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<int32> ShortcutNum;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<FVector> EnemyLoaction;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<float> EnemyHP;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<FVector> ResourceRock;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<FVector> ResourceTree;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<FVector> PickupStone;
	UPROPERTY(VisibleAnywhere, Category = "SlAi")
		TArray<FVector> PickupWood;
```
在`GameMode.h`下添加指针
```
private:
	//是否需要加载存档
	bool IsNeedLoadRecord;
	//游戏存档指针
	class USlAiSaveGame* GameRecord;
protected:
	//存档加载
	void LoadRecord();
	//给背包进行加载存档,这个函数一定要在第二帧再执行,否则快捷栏没初始化完成会崩溃
	void LoadRecordPackage();

```
在构造函数初始化
```
	//开始设置不需要加载存档
	IsNeedLoadRecord = false;
```
在`BeginPlay`运行函数`LoadRecord`

实现`LoadRecord`函数
```
	//如果RecordName为空,直接renturn
	if (SlAiDataHandle::Get()->RecordName.IsEmpty() || SlAiDataHandle::Get()->RecordName.Equals(FString("Default"))) return;
	//循环检测存档是否已经存在
	for (TArray<FString>::TIterator It(SlAiDataHandle::Get()->RecordDataList); It; ++It) {
		//如果有一个一样就直接设置为true,并且直接跳出循环
		if ((*It).Equals(SlAiDataHandle::Get()->RecordName)) {
			IsNeedLoadRecord = true;
			break;
		}
	}
	//如果需要加载,进行存档的加载,如果存档存在,进行加载
	if (IsNeedLoadRecord && UGameplayStatics::DoesSaveGameExist(SlAiDataHandle::Get()->RecordName, 0))
	{
		GameRecord = Cast<USlAiSaveGame>(UGameplayStatics::LoadGameFromSlot(SlAiDataHandle::Get()->RecordName, 0));
	}
	else {
		IsNeedLoadRecord = false;
	}
	//如果需要加载并且存档存在
	if (IsNeedLoadRecord && GameRecord)
	{
		//设置玩家位置和血量
		SPCharacter->SetActorLocation(GameRecord->PlayerLocation);
		SPState->LoadState(GameRecord->PlayerHP, GameRecord->PlayerHunger);
		//循环设置敌人
		int EnemyCount = 0;
		for (TActorIterator<ASlAiEnemyCharacter> EnemyIt(GetWorld()); EnemyIt; ++EnemyIt) {
			if (EnemyCount < GameRecord->EnemyLoaction.Num())
			{
				(*EnemyIt)->SetActorLocation(GameRecord->EnemyLoaction[EnemyCount]);
				(*EnemyIt)->LoadHP(GameRecord->EnemyHP[EnemyCount]);
			}
			else {
				//告诉这个敌人下一帧销毁
				(*EnemyIt)->IsDestroyNextTick = true;
			}
			++EnemyCount;
		}
		//循环设置岩石
		int RockCount = 0;
		for (TActorIterator<ASlAiResourceRock> RockIt(GetWorld()); RockIt; ++RockIt) {
			if (RockCount < GameRecord->ResourceRock.Num()) {
				(*RockIt)->SetActorLocation(GameRecord->ResourceRock[RockCount]);
			}
			else {
				//告诉这个资源下一帧销毁
				(*RockIt)->IsDestroyNextTick = true;
			}
			++RockCount;
		}
		//循环设置树木
		int TreeCount = 0;
		for (TActorIterator<ASlAiResourceTree> TreeIt(GetWorld()); TreeIt; ++TreeIt) {
			if (TreeCount < GameRecord->ResourceTree.Num()) {
				(*TreeIt)->SetActorLocation(GameRecord->ResourceTree[TreeCount]);
			}
			else {
				//告诉这个资源下一帧销毁
				(*TreeIt)->IsDestroyNextTick = true;
			}
			++TreeCount;
		}
		//循环设置拾取物品石头
		int StoneCount = 0;
		for (TActorIterator<ASlAiPickupStone> StoneIt(GetWorld()); StoneIt; ++StoneIt) {
			if (StoneCount < GameRecord->PickupStone.Num()) {
				(*StoneIt)->SetActorLocation(GameRecord->PickupStone[StoneCount]);
			}
			else {
				//告诉这个资源下一帧销毁
				(*StoneIt)->IsDestroyNextTick = true;
			}
			++StoneCount;
		}
		//循环设置拾取物品木头
		int WoodCount = 0;
		for (TActorIterator<ASlAiPickupWood> WoodIt(GetWorld()); WoodIt; ++WoodIt) {
			if (WoodCount < GameRecord->PickupWood.Num()) {
				(*WoodIt)->SetActorLocation(GameRecord->PickupWood[WoodCount]);
			}
			else {
				//告诉这个资源下一帧销毁
				(*WoodIt)->IsDestroyNextTick = true;
			}
			++WoodCount;
		}
	}
```
再Tick函数添加：
```
	//给背包加载存档,放在初始化背包上面是为了在第二帧再执行
	LoadRecordPackage();
```
实现`LoadRecordPackage`函数
```
	//如果背包没有初始化或者不用加载存档,直接返回
	if (!IsInitPackage || !IsNeedLoadRecord) return;
	if (IsNeedLoadRecord && GameRecord)
	{
		SlAiPackageManager::Get()->LoadRecord(&GameRecord->InputIndex, &GameRecord->InputNum, &GameRecord->NormalIndex, &GameRecord->NormalNum, &GameRecord->ShortcutIndex, &GameRecord->ShortcutNum);
	}
	//最后设置不用加载存档了
	IsNeedLoadRecord = false;
```

##32.2 保存存档与项目完结
1.**保存存档的实现**
在`SLAiEnemyCharacter.h`声明public函数
```
float GetHP();
```
实现：
```
	return HP;
```

在`SlAiPlayerState.h`声明public函数
```
	//保存血量和饥饿值到指定数据
	void SaveState(float& HPVal, float& HungerVal);
```
实现：
```
	HPVal = HP;
	HungerVal = Hunger;
```
在`SlAiPackageManager.h`声明public函数
```
	void SaveData(TArray<int32>& InputIndex, TArray<int32>& InputNum, TArray<int32>& NormalIndex, TArray<int32>& NormalNum, TArray<int32>& ShortcutIndex, TArray<int32>& ShortcutNum);
```
实现:
```
	for (int i = 0; i < InputContainerList.Num(); ++i)
	{
		InputIndex.Add(InputContainerList[i]->GetIndex());
		InputNum.Add(InputContainerList[i]->GetNum());
	}
	for (int i = 0; i < NormalContainerList.Num(); ++i) {
		NormalIndex.Add(NormalContainerList[i]->GetIndex());
		NormalNum.Add(NormalContainerList[i]->GetNum());
	}
	for (int i = 0; i < ShortcutContainerList.Num(); ++i) {
		ShortcutIndex.Add(ShortcutContainerList[i]->GetIndex());
		ShortcutNum.Add(ShortcutContainerList[i]->GetNum());
	}
```
在`SlAiDataHandle.h`声明public函数
```
void AddNewRecord();
```
实现：
```
	//将现在的存档名添加到数组
	RecordDataList.Add(RecordName);
	//更新json数据
	SlAiSingleton<SlAiJsonHandle>::Get()->UpdateRecordData(GetEnumValueAsString<ECultureTeam>(FString("ECultureTeam"), CurrentCulture), MusicVolume, SoundVolume, &RecordDataList);
```

在`GameMode`声明public函数
```
	//保存游戏
	void SaveGame();
```
然后在`SlAiGameHUD`的`BeginPlay`添加：
```
	//保存游戏事件绑定
	GameHUDWidget->GameMenuWidget->SaveGameDele.BindUObject(GM, &ASlAiGameMode::SaveGame);
```
并添加头文件
```
#include "SSlAiGameMenuWidget.h"
```
实现函数：
```
	//如果存档名是Default,不进行保存
	if (SlAiDataHandle::Get()->RecordName.Equals(FString("Default"))) return;
	//创建一个新的存档
	USlAiSaveGame* NewRecord = Cast<USlAiSaveGame>(UGameplayStatics::CreateSaveGameObject(USlAiSaveGame::StaticClass()));
	//对存档进行赋值
	//设置玩家位置和血量
	NewRecord->PlayerLocation = SPCharacter->GetActorLocation();
	SPState->SaveState(NewRecord->PlayerHP, NewRecord->PlayerHunger);
	//循环设置敌人
	for (TActorIterator<ASlAiEnemyCharacter> EnemyIt(GetWorld()); EnemyIt; ++EnemyIt)
	{
		NewRecord->EnemyLoaction.Add((*EnemyIt)->GetActorLocation());
		NewRecord->EnemyHP.Add((*EnemyIt)->GetHP());
	}
	//循环设置岩石
	for (TActorIterator<ASlAiResourceRock> RockIt(GetWorld()); RockIt; ++RockIt)
	{
		NewRecord->ResourceRock.Add((*RockIt)->GetActorLocation());
	}
	//循环设置树木
	for (TActorIterator<ASlAiResourceTree> TreeIt(GetWorld()); TreeIt; ++TreeIt) {
		NewRecord->ResourceTree.Add((*TreeIt)->GetActorLocation());
	}
	//循环设置拾取物品石头
	for (TActorIterator<ASlAiPickupStone> StoneIt(GetWorld()); StoneIt; ++StoneIt) {
		NewRecord->PickupStone.Add((*StoneIt)->GetActorLocation());
	}
	//循环设置拾取物品木头
	for (TActorIterator<ASlAiPickupWood> WoodIt(GetWorld()); WoodIt; ++WoodIt) {
		NewRecord->PickupWood.Add((*WoodIt)->GetActorLocation());
	}
	//获取背包数据
	SlAiPackageManager::Get()->SaveData(NewRecord->InputIndex, NewRecord->InputNum, NewRecord->NormalIndex, NewRecord->NormalNum, NewRecord->ShortcutIndex, NewRecord->ShortcutNum);
	//查看是否已经有存档存在
	if (UGameplayStatics::DoesSaveGameExist(SlAiDataHandle::Get()->RecordName, 0)) {
		//有的话先删除
		UGameplayStatics::DeleteGameInSlot(SlAiDataHandle::Get()->RecordName, 0);
	}
	//保存存档
	UGameplayStatics::SaveGameToSlot(NewRecord, SlAiDataHandle::Get()->RecordName, 0);
	//查看json是否已经有这个存档
	bool IsRecordExist = false;
	for (TArray<FString>::TIterator It(SlAiDataHandle::Get()->RecordDataList); It; ++It)
	{
		//只要有一个相同,就跳出
		if ((*It).Equals(SlAiDataHandle::Get()->RecordName)) {
			IsRecordExist = true;
			break;
		}
	}
	//如果存档不存在,让数据管理类添加存档到json
	if (!IsRecordExist) SlAiDataHandle::Get()->AddNewRecord();
```
然后就可以编译通过进入编辑器进行最后设置

2.**最后设置**
打开`MenuMap`地图，进入项目设置，把默认地图改为`MenuMap`
开始游戏就可以测试存档和读取的功能