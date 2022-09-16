---
layout: post
title: "Unreal Engine 5 —— 入门篇（一）"
description: ""
date: 2022-09-15
feature_image: images/2022.8.24/0.jpg
tags: [Unreal]
---

一直断断续续学了一点 UE，但是总感觉不像 Unity 那样很有把握可以实现未知的功能，主要还是不太习惯蓝图类的连连看机制，以及很多 UE C++ 类不知道，因此还是有必要从基础认真学习一下。

<!--more-->

## 一些知识点

- UPROPERTY -> 反射机制，定义属性的一个宏，能够被蓝图识别 https://docs.unrealengine.com/5.0/zh-CN/unreal-engine-uproperty-specifiers/ 常用的：VisibleAnywhere<所有属性窗口可见，但是无法编辑>、BlueprintReadOnly<蓝图中是只读的>、Category=""<在蓝图编辑器中显示的属性类别>、meta=(AllowPrivateAccess="true",BlueprintProtected="true")<标记位掩码>、EditAnywhere
- UStaticMeshComponent -> 静态网格体
- FORCEINLINE -> 内联函数，定义在函数之前，避免调用时的开销，提升执行效率，不可用于代码过长或者循环
- 在构造函数中创建变量
- CreateDefaultSubobject<>(TEXT("")) -> 创建对象，然后绑定根节点 RootComponent
- UFUNCTION -> 对蓝图可视化脚本图表公开，以便开发者从蓝图资源调用或扩展 https://docs.unrealengine.com/5.0/zh-CN/ufunctions-in-unreal-engine/ 常用的：BlueprintPure、、Category、BlueprintCallable
- 继承之后需要有一个 public 构造函数设置默认值
- mesh->SetSimulatePhysics(true) -> 设置 Mesh 的物理状态
- 要设置 Mesh 本身的物理属性需要点原 Mesh
- Component 需要提前在构造函数中创建，然后设置根组件 RootComponent，返回值要加上 class
- FVector 是向量
- BoxComponent -> 中心 Bounds.Origin，边界 Bounds.BoxExtent
- 随机点生成 UKismetMathLibrary::RandomPointInBoundingBox(origin, extent)
- TSubclassOf<class > 
- UWorld* const World = GetWorld()
- FActorSpawnParameters 参数，Owner = this，Instigator = GetInstigator()
- FRotator 旋转，y->Yaw，z->Pitch，x->Roll
- FMath::Frand() 随机数
- FActorSpawnParameters 传递给SpawnActor函数的可选参数结构体，设置 Owner = this，Instigator = GetInstigator()
- World->SpawnActor<>(WhatToSpwan, Location, Rotation)
- FTimerHandle 定时器
- GetWorldTimerManager().SetTimer(SpwanTimer, this, &class::function, SpawnDelay, false)
- UFUNCTION -> BlueprintNativeEvent 蓝图事件，可以被蓝图执行
- FString 字符串
- GetName() -> 获取名字
- UE_LOG(LogClass, Log, TEXT("%s"),string)
- override 重写
- Super::XX -> 调用父类的方法
- Destroy() 销毁
- USphereComponent 球形组件
- SetupAttachment() 绑定父组件
- TArray<AActor*> 数组
- CollectionSphere->GetOverlappingActors(Array) 球体附近的数组
- Cast<> 指针转换，检测是否转换成功
- IsPendingKill() -> 是否被销毁
- PlayerInputComponent->BindAction("Jump", IE_Released/IE_Pressed, this, &ACharacter::StopJumping) 绑定事件
- 关卡蓝图是一种专业类型的蓝图，用作关卡范围的全局事件图，常用的事件如 BeginPlay、Tick
- 移动性是针对光照的，静态的会默认影子不动，可移动的影子是实时计算出来的，固定的光属性是可以变的，但是影子仍然不会动
- BeginOverlap 开始重叠，处理碰撞
- AddActorWorldOffset 位移节点，UE 长度单位是 1cm
- A 代表继承自 Actor，--ACharacter -> APawn -> AActor，在子类中实现方法时，如果是重载函数，首先需要调用父类方法，如 Super::BeginPlay()
- 反射 在 C++ 代码和蓝图之间建立了某种联系，蓝图就可以直接调用变量、函数，UPROPERT()、UFUNCTION()，在 .generated.h 中定义了构建反射的一些方法
- UCLASS(Blueprintable) 使 C++ 类能够生成蓝图，UPROPERTY(BlueprintReadWrite->蓝图能够读写, Category=""->设置分类, BlueprintReadOnly->蓝图只读)，UFUNCTION(BlueprintCallable->蓝图能够调用,Category=""->设置分类)
- UE_LOG()
- UPROPERTY(VisibleAnywhere->所有地方可见) UStaticMeshComponent* -> 静态网格体组件  CreateDefaultSubobject<UStaticMeshComponent>(TEXT("CustomStaticMesh")) -> 创建子物体
- 可以使用 Snapping 将物体吸附在地面上
- Get Actor Up Vector -> 获取角色的上方向，Forward -> 前 x，Right -> 右 y，Up -> 上 z
- struct FVector 三维向量
- SetActorLocation() 设置角色位置
- EditInstanceOnly -> 可以在实例中编辑，EditDefaultsOnly -> 蓝图窗口可修改
- 运行之后可以按 Shift+F1 让鼠标出来，鼠标不能移动也可以按 F8 解除控制
- 在蓝图中的 Physics 菜单下面，开启 Simulate Physics 便可以开启物理模拟
- 碰撞中重叠可以穿透，阻挡就是挡住
- AddLocalOffset()，sweep，HitResult 当阻挡 bBlockingHit 为 true
- FString::SanitizeFloat() 将 float 转化为 string
- 按下 ` 键可以出来命令行，show collision 显示碰撞
- Collision 碰撞可以设置碰撞类型
- FRotater() roll x pitch y yaw z AddActorLocalOffset() AddActorWorldRotation()
- StaticMesh AddTorque() 扭矩 AddForce() 力 约束可以锁定旋转
- FMath 数学函数 FRand() 返回 0~1 随机数 FRandRange() Sin()
- 删除 Binaries 文件夹和源代码源文件才能把 C++ 代码删干净，并且重新生成项目
- Pawn 继承自 Actor，比 Actor 多了输入控制 SetupPlayerInputComponent，根组件 RootComponent = CreateSubobject<USceneComponent>(TEXT())，GetRootComponent() 获取根组件，MeshComponent->SetupAttachment() 组件绑定，SetRelativeLocation()、SetRelativeRotation()
- 在游戏模式中设置 Default Pawn，则会把对应 Pawn 在 PlayerStart 处生成，然后在世界场景设置中选择我们的游戏模式
- AutoPossessPlayer = EAutoReceiveInput::Player0 设置由哪个玩家控制，让默认玩家直接拥有该角色
- PlayerInputComponent->BindAxis("MoveForward", this, &AMyPawn::MoveForward); 将 "MoveForward" 与实际的方法 &AMyPawn::MoveForward 产生联系，而键盘与 "MoveForward" 对应的设置需要在项目设置的输入中设置，Axis Mappings
- USphereComponent 球形组件，用于设置碰撞，SetSphereRadius、SetCollisionProfileName(TEXT())、SetHiddenInGame()
- SetRootComponent() 设置根组件
- 直接通过 Mesh 资源路径设置 Mesh，static ConstructorHelper::FObjectFinder<UStaticMesh>，然后将 SetStaticMesh
- USpringArmComponent 弹簧臂，TargetArmLength、bEnableCameralag、CameraLagSpeed，SetAttachment(SpringArmComp, USpringArmComponent::SocketName)、bUsePawnControlRotation
- UPawnMovementComponent 运动组件，TickComponent，要设置 UpdateComponent、PawnOwner、ShouldSkipUpdate(DeltaTime)，SlideAlongSurface 沿着物体表面滑动，AddMovementInput，AddInputVector
- 迁移可以让一个项目的资源转出去
- Character 角色 bUseControllerRotationPitch、bUseControllerRotationYaw、bUseControllerRotationRoll，GetCharacterMovement()->RotationRate、JumpZVelocity、AirControl、bOrientRotationToMovement
- AddMovementInput FRotationMatrix .GetUnittAxis(EAxis::X)
- BindAction(IE_Pressed, IE_Released) Action 有按下和松开
- 创建动画蓝图后需要在 Mesh 的 Animation 绑定，动画蓝图一般需要添加状态机，然后创建 Animinstance 控制动画播放逻辑，动画混合空间可以根据参数值设置动画，transition 条件中使用 timeRemaining(ratio) 动画还剩百分之几可以用于判断下一个动画
- TryGetPawnOwner() 获取主角，GetVelocity()


## 小结
  
  

## References

- [UE4 C++ 入门（无参考项目）[1/2]](https://www.bilibili.com/video/BV1RE411d7J8)