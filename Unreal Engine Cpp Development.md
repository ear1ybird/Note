# Unreal Engine Cpp Development

[TOC]

## Class Hierarchy

* Object
* Actor
* Pawn
* Character
* MyCharacter

## Class Prefixes

* U - Classes Deriving from Uobject
* A - deriving from Actor
* F - Structs
* E -Enums
* I - Interfaces

### Examples: 

* Acharacter(Class)

* FHitResult(Struct)

* EDirection(Enum)

## Unreal Property System

### Unreal Macros

UPROPERTY, UFUNCTION, UCLASS, USTRUCT, UENUM 

#### UPERPORTY

* EditAnywhere - edit in BP editor per-instance in level
* VisibleAnywhere - read-only in editor and level (use for components)
* EditDefaultsOnly - hide varible per-instance, edit in BP editor only
* VisibleDefaultsOnly - read only access for variable, only in BP editor (uncommom)



* BlueprintReadOnly - read only in the Blueprint scripting (does not affect 'details' panel)
* BlueprintReadWrite - allow only editing of instance (eg. when placed in level)



* Category = " " -display only for detail panels and blueprint comtext menu

### Marking up C++

* Editor & Blueprints Access
* Network behavior
* Memory Management

### Handles Boilerplate for us

## PlayerController

### Controller.h

Shared base for AI and players,not visually represented in world.

* Has 'ControlRotation' (for camera use etc.), separate from Pawn's Rotation

  **ControlRotation可以通过SpringArmComp传递给摄影机**

  ![image-20220621224143910](assets/image-20220621224143910.png)

  **ControlRotation可以传递给Pawn**

  ![image-20220621222829351](assets/image-20220621222829351.png)

  

* PlayerController.h

  One per player, persists for entire session (or until level changes)

  'possess' Pawn and forwards input from player

  Pawn can 'die' and destroyed while PlayerController persists

* AIController.h

  One per AI, holds the 'logic' that controls the Pawn
  
## Collision & Physics

  * Collision Channels
  * Object Types
  * Reactions: Block, Overlap, Ignore

  ### Collision Event

  ![image-20220622193757588](assets/image-20220622193757588.png)

* Generate Overlap Events：自身是否生成Overlap Events。**只在两物体同时开启此选项时才能生成Overlap Events。**
* Simulation Generates Hit Events：自身是否生成Hit Events。**此选项只再两个物体都开启Simulation Physics时有效**。

## Interface

创建Unreal Interface后，会生成两个Class：

```c++
// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class USGamePlayInterface : public UInterface
{
	GENERATED_BODY()
};

class UECPPACTIONROGUELIKE_API ISGamePlayInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
	UFUNCTION(BlueprintNativeEvent)
	void Interact(APawn* InstigatorActor);
};
```

* **其中ISGamePlayInterface用于继承，USGamePlayInterface用于Implements函数判断是否继承**

* UFUNCTION(BlueprintNativeEvent)：可在C++中实现、也能在蓝图中实现
* UFUNCTION(BlueprintImplementableEvent)：只能在蓝图中实现

### 在蓝图中调用 BlueprintNativeEvent 接口

蓝图类是Cpp类的子类，所以在蓝图中实现的接口会重写Cpp接口。

蓝图类可以调用Cpp类的接口：![image-20220625175730760](assets/image-20220625175730760.png)

## UE_LOG

### 常见用法

```c++
UE_LOG(LogTemp, Warning, TEXT("Hello"));

UE_LOG(LogTemp, Warning, TEXT("The Actor's name is %s"), *YourActor->GetName());

UE_LOG(LogTemp, Warning, TEXT("The boolean value is %s"), ( bYourBool ? TEXT("true") : TEXT("false") ));

UE_LOG(LogTemp, Warning, TEXT("The integer value is: %d"), YourInteger);

UE_LOG(LogTemp, Warning, TEXT("The float value is: %f"), YourFloat);

UE_LOG(LogTemp, Warning, TEXT("The vector value is: %s"), *YourVector.ToString());

UE_LOG(LogTemp, Warning, TEXT("Current values are: vector %s, float %f, and integer %d"), *YourVector.ToString(), YourFloat, YourInteger);
```

**其中字符串的重载运算符 \*，用户将FString转换为期望类型（如TChar\* 字符串数组）**

### UE_LOG 常用函数

```c++
//获取Name而不关心是否是空指针
FString GetNameSafe(AACtor* Actor);

//获取启动游戏后的计时器
float GetWorld()->TimeSeconds; 
```

### DrawDebugString()

用于在世界空间打印字符串进行调试

## Trace

* Trace类型：LineTrace、SweepTrace
* Trace数量：TraceSingle、TraceMulti
* Trace方式：ByChannel、ByObjectType、ByProfile

### ByChannel

**这根射线的 Collision Type 是 WorldStatic**，所有与 WorldStatic 类型为Block的，都会挡住射线

### ByObjectType

设置其他物体的类型

```c++
FCollisionObjectQueryParams ObjectQueryParams;
ObjectQueryParams.AddObjectTypesToQuery(ECC_WorldDynamic);
```

## Assert断言

[断言 | 虚幻引擎文档 (unrealengine.com)](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/Assertions/)

Assert可以在开发期间帮助检测无效运行条件，但每次检查会使得效率十分低下，Assert的关键特效是不存在于发布代码中。

### Check

当第一个参数得出的值为false时，此族系的成员会停止执行，且**默认不会在发布版本中运行**。

### Verify

即便在禁用Check宏的版本中，Verify宏也会计算其表达式的值。这意味着仅当该表达式需要独立于诊断检查之外运行时，才应使用Verify宏。举例而言，若某个函数执行操作，然后返回 `bool` 来说明该操作是否成功，则应使用Verify而非Check来确保该操作成功。因为在发布版本中**Verify将忽略返回值，但仍将执行操作**。而Check在发布版本中根本不调用该函数，所以行为才会有所不同。

### Ensure

若Ensure宏的表达式计算得出的值为false，**引擎将通知崩溃报告器，但仍会继续运行**。为避免崩溃报告器收到太多通知，**Ensure宏在每次引擎或编辑器会话中仅报告一次**。

EnsureAlways宏将会每次都触发中断。

# 踩坑记录

1. 创建组件不会默认设置为根组件的子物体。在构造函数中使用SetupAttachment，在运行时使用AttachToComponent。
