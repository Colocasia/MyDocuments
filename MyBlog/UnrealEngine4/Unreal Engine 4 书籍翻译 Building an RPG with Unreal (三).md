[TOC]

# **Unreal Engine 4 书籍翻译 Building an RPG with Unreal (三)**#

好记性不如烂笔头啊，还是记录一下!<br>
自己翻译的书，可能翻译的不好，大家见谅。<br>
欢迎大家指出翻译错误的地方以便修正

----------

## **<font color=#191970 size=5>第3章 探索和战斗</font> ** ##

我们完成一个我们游戏的设计和我们游戏的一个虚幻项目设置，现在是时候编写我们实际的游戏代码了。
在这一章，我们会定义我们的游戏数据来创造一个可以在世界中移动的角色和游戏的基础战斗原型。在这一章我们将进行下列主题：

- <font color=#BD63C5 size=3>创建玩家</font>
- <font color=#BD63C5 size=3>定义所有类、角色、敌人</font>
- <font color=#BD63C5 size=3>持续追踪活动的伙伴成员</font>
- <font color=#BD63C5 size=3>创建一个基本的回合制战斗引擎</font>
- <font color=#BD63C5 size=3>在屏幕上触发一个游戏</font>

----------

### **<font color=#191970 size=4>创建角色</font> ** ###

我们要做的首要事情是创建一个新的Pawn类。在虚幻引擎中，Pawn是一个角色的表现形式，他是一个可以处理运动，物理和渲染的角色。
现在如何让我们的Pawn角色去工作。一个玩家分为两个部分：<br>
&emsp;&emsp;&ensp;1.Pawn——他负责处理责处理运动，物理，和渲染。<br>
&emsp;&emsp;&ensp;2.Player Controller——是负责将玩家的输入进行处理，可以使得Pawn像玩家所想的那样行动。<br>
此外，我们执行一层分离，使Pwan类实现一个名为IControllableCharacter的接口。这不是绝对必要的，但确实有助于防止不同类需要太多了解对方（例如，任何其他的Actor可以实现IControllableCharacter接口，我们的玩家也可以同样的控制那些Actor）

----------

### **<font color=#191970 size=4>接口</font> ** ###

所以首先，我们会研究这个接口。我们从创建一个Actor的派生类开始，正如我们在前一章做。命名为 ControllableCharacter。虚幻引擎生成代码文件后，打开 ControllableCharacter.h，并改变它
为下面的代码︰

- ControllableCharacter.h

``` cpp

#pragma once
#include "Object.h"
#include "ControllableCharacter.generated.h"

UINTERFACE()
class RPG_API UControllableCharacter : public UInterface
{
    GENERATED_UINTERFACE_BODY()
};

class RPG_API IControllableCharacter
{
    GENERATED_IINTERFACE_BODY()
    virtual void MoveVertical( float Value );
    virtual void MoveHorizontal( float Value );
};

```

让我们逐句分析下这些代码都做了什么？
在虚幻中，接口有两个部分︰UInterface类和实际类。这两个类，结合虚幻的宏系统，允许您提供接口类转换宏 （这我们稍后将讨论）去转换到Actor实现的接口。

- UInterface类有U前缀（所以在这种情况下，它是UControllableCharacter）只包含一行<font color=#BD63C5 size=3>GENERATED_UINTERFACE_BODY()</font>。
- 实际的接口类具有I前缀（所以在这种情况下，它是IControllableCharacter），其中有一行<font color=#BD63C5 size=3>GENERATED_IINTERFACE_BODY()</font>，还有实际定义的接口（在这里，我们定义的MoveVertical和MoveHorizontal的方法）。

接下来，打开ControllableCharacter.cpp，将其更改为下面的代码

- ControllableCharacter.cpp

``` cpp

#include "RPG.h"
#include "ControllableCharacter.h"
UControllableCharacter::UControllableCharacter( const class FObjectInitializer& ObjectInitializer )
  : Super( ObjectInitializer )
{
}

void IControllableCharacter::MoveVertical( float Value )
{
}

void IControllableCharacter::MoveHorizontal( float Value )
{
}

```

在这里，我们只定义了 UControllableCharacter 类的构造函数和 MoveHorizontal 和 MoveVertical 的默认实现。

----------

### **<font color=#191970 size=4>PlayerController</font> ** ###

接下来，我们要创建PlayerController。PlayerController的作用如前所述，是将玩家输入进行转换，从而实际控制角色的行动。
创建一个新类，选择PlayerController作为基类。它的名字RPGPlayerController。
打开生成的RPGPlayerController.h文件并在类中添加以下代码：

- RPGPlayerController.h

``` cpp

protected:
    void MoveVertical( float Value );
    void MoveHorizontal( float Value );
    virtual void SetupInputComponent() override;

```

前两种方法MoveVertical和MoveHorizontal我们已经定义了，是我们将用来侦听玩家输入的两个方法。稍后我们将建立当玩家按下动作键或摇杆时调用这两个方法。
最后一个方法SetupInputComponent，是一个重写的内置方法。顾名思义我们会在这方法中设置输入组件。
接下来，打开RPGPlayerController.cpp并添加以下代码︰

- RPGPlayerController.cpp

``` cpp

void ARPGPlayerController::MoveVertical( float Value )
{
    IControllableCharacter* pawn = Cast<IControllableCharacter>( GetPawn() );
    if( pawn != NULL )
    {
        pawn->MoveVertical( Value );
    }
}
void ARPGPlayerController::MoveHorizontal( float Value )
{
    IControllableCharacter* pawn = Cast<IControllableCharacter>( GetPawn() );
    if( pawn != NULL )
    {
        pawn->MoveHorizontal( Value );
    }
}
void ARPGPlayerController::SetupInputComponent()
{
    if( InputComponent == NULL )
    {
        InputComponent = ConstructObject<UInputComponent>(UInputComponent::StaticClass(), this, TEXT( "PC_InputComponent0" ) );
        InputComponent->RegisterComponent();
    }
    InputComponent->BindAxis( "MoveVertical", this, &ARPGPlayerController::MoveVertical );
    InputComponent->BindAxis( "MoveHorizontal", this, &ARPGPlayerController::MoveHorizontal );
    this->bShowMouseCursor = true;
}

```

在这里，我们实现在头文件中定义的方法。让我们来看看它们都做了些什么？
MoveHorizontal和MoveVertical这两个方法是几乎完全相同，所以我们只需要要看看 MoveHorizontal。
首先，我们使用下面的行︰

``` cpp

IControllableCharacter* pawn = Cast<IControllableCharacter>( GetPawn() );

```

这个方法获取一个PlayerController是当前正在控制中的Pawn指针，然后投射到我们定义Pawn的IControllableCharacter里的接口。
接下来，我们检查指针是否为null，如果不空，我们就调用MoveHorizontal方法来使用A键和D键（如果是MoveVertical则使用W键和S键）控制Pawn，取值范围从-1到1（例如，在使用MoveHorizontal方法时，如果玩家按下A键，则值将会为-1。如果玩家按下D键，则值将会为1。如果玩家不按下任何按键，则值将会为0）
在SetupInputComponent这个方法中，我们首先看看下面的代码：

``` cpp

if( InputComponent == NULL )
{
    InputComponent = ConstructObject<UInputComponent>(UInputComponent::StaticClass(), this, TEXT( "PC_InputComponent0" ) );
}

```

基本上，如果这里没有附加任何输入控件，我们构造一个新的UInputComponent类的实例（通过ConstructObject宏，我们传入的参数分别是类型，构造的哪个类，这个组件要附加到的Actor，新组件的名称）
现在，我们有了一个输入组件，我们用它绑定我们的运动轴：

``` cpp

InputComponent->BindAxis( "MoveVertical", this, &ARPGPlayerController::MoveVertical );

```

BindAxis方法设置了一个函数引用一遍在使用输入轴的值时来调用。在前面的代码行，我们调用BindAxis传递的参数为轴的名称，一个指向调用函数Actor的指针，一个Actor用来处理输入的方法的引用
最后，我们设置bShowMouseCursor为true，所以那虚幻不会隐藏鼠标光标。

----------

### **<font color=#191970 size=4>The Pawn</font> ** ###

现在现在，让我们来创建实际的Pawn。
创建一个新类并选择Character作为他的父类。取名为RPGCharacter，打开RPGCharacter.h，并在类定义中更改代码为下面的代码：

- RPGCharacter.h

``` cpp

UCLASS()
class RPG_API ARPGCharacter : public ACharacter, public IControllableCharacter
{

    GENERATED_BODY()
    ARPGCharacter( const class FObjectInitializer& ObjectInitializer );

public:

    virtual void MoveVertical( float Value );
    virtual void MoveHorizontal( float Value );
};

```

首先，我们用我们的新类实现IControllableCharacter里的接口。在类中，我们也定义了构造函数、MoveVertical和MoveHorizontal的方法（这是必须要实现的IControllableCharacter的接口）。
接下来，打开RPGCharacter.cpp并添加以下代码︰

- RPGCharacter.cpp

``` cpp

ARPGCharacter::ARPGCharacter( const class FObjectInitializer &ObjectInitializer )
    : Super( ObjectInitializer )
{
    bUseControllerRotationYaw = false;
    GetCharacterMovement()->bOrientRotationToMovement = true;
    GetCharacterMovement()->RotationRate = FRotator( 0.0f, 0.0f, 540.0f );
    GetCharacterMovement()->MaxWalkSpeed = 400.0f;
}

void ARPGCharacter::MoveVertical( float Value )
{
    if( Controller != NULL && Value != 0.0f )
    {
        const FVector moveDir = FVector( 1, 0, 0 );
        AddMovementInput( moveDir, Value );
    }
}

void ARPGCharacter::MoveHorizontal( float Value )
{
    if( Controller != NULL && Value != 0.0f )
    {
        const FVector moveDir = FVector( 0, 1, 0 );
        AddMovementInput( moveDir, Value );
    }
}

```

虚幻中的Character有一些内置的运动属性。在构造函数中，我们设置运动组件的一些默认值（默认情况下，Character旋转到面向运动方向的速度为540单位/秒，最大运动速度为400单位/秒）
我们还用构造一个运动向量传递给AddMovementInput，来实现MoveHorizo​​ntal和MoveVertical方法。

----------

### **<font color=#191970 size=4>游戏模式类</font> ** ###

现在，为了使用这些类，我们需要建立了一类新的游戏模式。游戏模式可以指定默认使用的Pwan和PlayerController，我们还可以使用游戏模式的蓝图来修改这些默认的Pawn和PlayerController。
创建一个新类，选择GameMode作为新类的父类。并将类命名为RPGGameMode。
打开RPGGameMode.h并更改类的定义，使用以下代码︰

- RPGGameMode.h

``` cpp

UCLASS()
class RPG_API ARPGGameMode : public AGameMode
{
    GENERATED_BODY()
    ARPGGameMode( const class FObjectInitializer& ObjectInitializer );
};

```

正如我们前面已经做过的，我们只需要定义的CPP文件中的构造函数来实现。
我们现在就在RPGGameMode.cpp中实现构造函数：

- RPGGameMode.cpp

``` cpp

#include "RPGPlayerController.h"
#include "RPGCharacter.h"

ARPGGameMode::ARPGGameMode( const class FObjectInitializer &ObjectInitializer )
    : Super( ObjectInitializer )
{
    PlayerControllerClass = ARPGPlayerController::StaticClass();
    DefaultPawnClass = ARPGCharacter::StaticClass();
}

```

这里，我们包含RPGPlayerController.h和RPGCharacter.h这两个头文件，这样我们就可以引用这些类。然后，在构造函数中，我们将设置这些类作为默认的PlayerController和Pawn。
现在，如果你编译此代码，你必须去设置你的工程的默认游戏模式为你的新建立的游戏模式。要做到这一点，请到<font color=#BD63C5 size=3>编辑->项目设置</font>，找到默认游戏模式选项框，展开默认游戏模式下拉菜单，并选择RPGGameMode。
然而，我们不一定要直接使用此类。相反，如果我们使用蓝图，我们可以将游戏模式的属性公开的，就可以在蓝图中更新这些公开的属性。
所以，让我们创造一个新的蓝图，命名为DefaultRPGGameMode，让它继承自RPGGameMode：

![GameMode](http://img.blog.csdn.net/20161114111241060)

如果你打开这个新的蓝图，导航到默认值选项卡，你可以修改默认的Pawn，HUD，PlayerController以及更多的设置：

![GameMode](http://img.blog.csdn.net/20161114113145899)

然而，在我们测试我们新的Pawn和PlayerController之前还有一个额外的步骤。如果你现在运行游戏，你会看不见Pawn。事实上，运行时就像什么都没有发生一样。因为我们需要给我们的Pawn设置一个模型和必须设置一个摄像机跟随我们的Pawn。

----------

### **<font color=#191970 size=4>添加模型</font> ** ###

现在，我们只需要导入第三人称示例中的蓝色角色原型。要做到这一点，请创建一个新的基于第三人称游戏的示例，并将以下内容迁移：

- <font color=#BD63C5 size=3>HeroTPP</font>
- <font color=#BD63C5 size=3>HeroTPP_AnimBlueprint</font>
- <font color=#BD63C5 size=3>HeroTPP_Skeleton</font>
- <font color=#BD63C5 size=3>IdleRun_TPP</font>

通过以下步骤将这些项目迁移到RPG项目中：

1. <font color=#BD63C5 size=3>选中这些资源</font>
2. <font color=#BD63C5 size=3>右键单击其中任何一个资源，并选择迁移</font>
3. <font color=#BD63C5 size=3>单击确定</font>
4. <font color=#BD63C5 size=3>在RPG项目中保存这些资源的文件夹</font>
5. <font color=#BD63C5 size=3>单击确定</font>

现在，使用你RPG项目中的<font color=#BD63C5 size=3>HeroTPP</font>模型，让我们为我们的Pawn创建一个新的蓝图。创建一个新的蓝图并选择<font color=#BD63C5 size=3>RPGCharacter</font>作为父类，取名为FieldPlayer。<br>
首先，展开**<font color=#BD63C5 size=3>Mesh</font>**选项并选择**<font color=#BD63C5 size=3>HeroTPP</font>**作为Pawn的骨骼。<br>
然后，展开**<font color=#BD63C5 size=3>Animation</font>**选项并选择HeroTPP_AnimBlueprint作为Pawn的动作。<br>
最后，打开你的游戏模式的蓝图，选择新的FieldPlayer作为你的默认Pawn。<br>
现在，你可以看见你的角色，并且在移动时可以播放一个跑动的动作。<br>
然而，摄像机不会跟随这个角色。我们会通过创建一个自定义的摄像机来解决这个问题。

----------

### **<font color=#191970 size=4>创建摄像机组件</font> ** ###

首先，创建一个新类，选择CameraComponent作为父类，命名为RPGCameraComponent。<br>
![RPGCamera](http://img.blog.csdn.net/20161115085239627)
接着，带开RPGCameraComponent.h并在类定义中使用如下代码：

- RPGCameraComponent.h

``` cpp

UCLASS( meta = ( BlueprintSpawnableComponent ) )
class RPG_API URPGCameraComponent : public UCameraComponent
{
    GENERATED_BODY()

public:

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CameraProperties)
    float CameraPitch;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CameraProperties)
    float CameraDistance;

    virtual void GetCameraView( float DeltaTime, FMinimalViewInfo &DesiredView ) override;
};

```

让我们看看这一切意味着什么。<br>
首先，在**<font color=#BD63C5 size=3>UCLASS</font>**宏中，我们添加了这行：
``` cpp

{
UCLASS( meta = ( BlueprintSpawnableComponent ) )    
}

```

这使得我们可以再Pawn蓝图中添加我们的自定义组件。<br>
接着，我们定义了两个字段，CameraPitch和CameraDistance。

- <font color=#BD63C5 size=3>CameraPitch控制摄像机视角</font>
- <font color=#BD63C5 size=3>CameraDistance控制摄像机距离</font>

并将这两个字段添加到CameraProperties这个类别下，这个字段的属性为EditAnywhere和BlueprintReadWrite<br>
最后，我们重写了负责计算摄像机位置，旋转和其他各种属性的GetCameraView函数。当具有此组件的pawn设置为当前视图目标时，虚幻会调用这个函数去定位游戏摄像机。<br>
接下来，打开RPGCameraComponent.cpp并添加以下代码：

- RPGCameraComponent.cpp

``` cpp

void URPGCameraComponent::GetCameraView( float DeltaTime, FMinimalViewInfo& DesiredView )
{
    UCameraComponent::GetCameraView( DeltaTime, DesiredView );
    DesiredView.Rotation = FRotator( CameraPitch, 0.0f, 0.0f );
    if( APawn* OwningPawn = Cast<APawn>( GetOwner() ) )
    {
        FVector location = OwningPawn->GetActorLocation();
        location -= DesiredView.Rotation.Vector() * CameraDistance;
        DesiredView.Location = location;
    }
}

```

此函数将覆盖内置的GetCameraView函数。<br>
首先，它调用基类的GetCameraView函数来确保正确的设置了DesiredView。<br>
然后，它从CameraPitch创建了一个FRotator并将其分配给DesiredView的旋转。<br>
最后，它尝试将其所有者转换为APawn。如果OwningPawn不为空，则获取OwningPawn的位置，并减去摄像机的前向向量与CameraDistance的距离。然后将结果分配给DesiredView的位置。<br>
接着，我们需要给我们的Pawn添加摄像机组件，打开你前面章节创建的蓝图Pawn。在<font color=#BD63C5 size=3>Components</font>选项卡中，点击<font color=#BD63C5 size=3>Add Component</font>。当你搜索RPGCamera，你刚才创建的自定义组件会出现在列表中。<br>
在你添加了RPGCameraComponent组件后，滑动你的<font color=#BD63C5 size=3>Details</font>面板直到你看见了<font color=#BD63C5 size=3>Camera Properties</font>属性框。在这里，你可以输入任何你喜欢的值可以改变相机的俯仰程度和距离。但是开始你可以设置为50的俯仰值和600的距离值。<br>
现在，当你运行游戏，摄像机可以在俯视图中跟踪玩家。<br>
现在，我们有了一个可以探索游戏世界的角色，让我们来看看定义角色和伙伴成员。

----------

### **<font color=#191970 size=4>定义角色和敌人</font> ** ###

在上一章节中，我们介绍了如何使用数据表来导入自定义数据。在那之前，我们决定了数据如何在战斗中发挥。现在我们要结合那些来定义我们的游戏的角色，类别和遭遇敌人。

----------

### **<font color=#191970 size=4>类别</font> ** ###

回顾第一章的内容，在虚幻中设计一个RPG，我们设定了我们的角色有一下属性：

 - <font color=#BD63C5 size=3>生命值</font>
 - <font color=#BD63C5 size=3>最大生命值</font>
 - <font color=#BD63C5 size=3>魔法值</font>
 - <font color=#BD63C5 size=3>最大魔法值</font>
 - <font color=#BD63C5 size=3>攻击力</font>
 - <font color=#BD63C5 size=3>防御</font>
 - <font color=#BD63C5 size=3>幸运</font>

其中，我们可以先不定义生命值和魔法值，因为这两个值会在比赛期间变化。其他值都是角色预定义的基础值，这些就是我们将在数据表中定义的数据。如第一张"RPG入门"所述，我们也需要存储的值是50级（最高等级）。角色将有一些初始能力，还可以在升级时学习一些能力。<br>
我们将在角色类的电子表格中定义这些属性，以及类别的名称。所以我们的角色类表格结构看起来像下面这样：

- <font color=#BD63C5 size=3>名称（字符串）</font>
- <font color=#BD63C5 size=3>初始最大生命值（整数）</font>（1级时）
- <font color=#BD63C5 size=3>最终最大生命值（整数）</font>（50级时）
- <font color=#BD63C5 size=3>初始最大魔法值（整数）</font>（1级时）
- <font color=#BD63C5 size=3>最终最大魔法值（整数）</font>（50级时）
- <font color=#BD63C5 size=3>初始攻击力（整数）</font>（1级时）
- <font color=#BD63C5 size=3>最终攻击力（整数）</font>（50级时）
- <font color=#BD63C5 size=3>初始防御力（整数）</font>（1级时）
- <font color=#BD63C5 size=3>最终防御力（整数）</font>（50级时）
- <font color=#BD63C5 size=3>初始幸运值（整数）</font>（1级时）
- <font color=#BD63C5 size=3>最终幸运值（整数）</font>（50级时）
- <font color=#BD63C5 size=3>初始能力列表（字符串数组）</font>（1级时）
- <font color=#BD63C5 size=3>学习能力列表（字符串数组）</font>
- <font color=#BD63C5 size=3>学习能力等级（整数数组）</font>

能力字符串数组将包含能力的ID（虚幻中是保留字段）。还有两列来存储学习能力信息——一列为所有可以学习的能力的ID数组，另一列为学习这些能力所需要的等级数组。<br>

在创造游戏的过程中，你应该考虑编写一个自定义工具来帮助管理数据，这样可以减少人为错误。但是，编写类似的工具不属于本书的范围。<br>

现在，我们不应该先为这些属性去创建电子表格，其实我们应该先在Unreal里创建类，再去创建数据表格。原因是，在填写数据时，没有好的文档记载怎么样在数据表的单元格中指定数组。然而在虚幻编辑器中我们可以编辑数组。所以我们简单的创建表格，然后使用虚幻的数组编辑器编辑。<br>

首先，像前面所做的一样，创建一个新类，它从哪里继承不是很重要。所以我们选择Actor类，命名为FCharacterClassInfo。<br>

打开FCharacterClassInfo.h，并使用以下代码替换类的定义：

- FCharacterClassInfo.h

``` cpp

USTRUCT( BlueprintType )
struct FCharacterClassInfo : public FTableRowBase
{
    GENERATED_USTRUCT_BODY()
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    FString Class_Name;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 StartMHP;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 StartMMP;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 StartATK;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 StartDEF;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 StartLuck;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 EndMHP;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 EndMMP;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 EndATK;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 EndDEF;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    int32 EndLuck;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    TArray<FString> StartingAbilities;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    TArray<FString> LearnedAbilities;
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "ClassInfo" )
    TArray<int32> LearnedAbilityLevels;
};

```

你应该很熟悉大部分的内容，但是最后三个字段你可能不认识。他们都是<font color=#BD63C5 size=3>TArray</font>类型，这是虚幻提供的动态数组类型。从根本上说，<font color=#BD63C5 size=3>TArray</font>和C++的数组不同，他可以动态的添加元素和移除元素<br>

再次编译代码后，你可以通过右键点击<font color=#BD63C5 size=3>Content Browser</font>，然后选择<font color=#BD63C5 size=3>Create Advanced Asset</font>-><font color=#BD63C5 size=3>Miscellaneous</font>-><font color=#BD63C5 size=3>Data Table</font>，来创建一个数据表格。然后，在下拉列表中选择<font color=#BD63C5 size=3>Character Class Info</font>，为你的数据表起个名字，然后双击打开它，你会看到下面的画面：

![DataTable](http://img.blog.csdn.net/20161115111934590)

如果<font color=#BD63C5 size=3>Row Editor</font>窗格为空的，你可能需要重启你的虚幻编辑器<br>

要添加新条目，请点击<font color=#BD63C5 size=3>Add</font>按钮。通过向<font color=#BD63C5 size=3>Rename</font>字段里输入文字，然后点击Enter键来给新条目命名。<br>

添加条目后，可以在<font color=#BD63C5 size=3>Data Table</font>窗格中选择该条目，然后在<font color=#BD63C5 size=3>Row Editor</font>穿个对它的属性进行编辑。<br>

我们在列表中添加一个Soldier类。我们将它命名为S1（我们将使用它来引用
来自其他数据表的角色类），它具有以下属性：

- <font color=#BD63C5 size=3>Class name:</font> Soldier
- <font color=#BD63C5 size=3>Start MHP:</font> 100
- <font color=#BD63C5 size=3>Start MMP:</font> 100
- <font color=#BD63C5 size=3>Start ATK:</font> 5
- <font color=#BD63C5 size=3>Start DEF:</font> 0
- <font color=#BD63C5 size=3>Start Luck:</font> 0
- <font color=#BD63C5 size=3>End MHP:</font> 800
- <font color=#BD63C5 size=3>End MMP:</font> 500
- <font color=#BD63C5 size=3>End ATK:</font> 20
- <font color=#BD63C5 size=3>End DEF:</font> 10
- <font color=#BD63C5 size=3>End Luck:</font> 10
- <font color=#BD63C5 size=3>Starting abilities:</font> 现在为空
- <font color=#BD63C5 size=3>Learned abilities:</font> 现在为空
- <font color=#BD63C5 size=3>Learned ability levels:</font> 现在为空

----------

### **<font color=#191970 size=4>角色</font> ** ###

让我们来看看角色类的定义，大部分的战斗相关数据已经在character类别里定义了，角色本身会变的非常简单。事实上，现在我们的角色只需要定义两个事情：角色名称和角色引用的类别ID<br>

首先，让我们先来看看角色数据的头文件，FCharacterInfo.h：

- FCharacterInfo.h

``` cpp

USTRUCT(BlueprintType)
struct FCharacterInfo : public FTableRowBase
{
    GENERATED_USTRUCT_BODY()
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "CharacterInfo" )
    FString Character_Name;
    UPROPERTY( BlueprintReadOnly, EditAnywhere, Category = "CharacterInfo" )
    FString Class_ID;
};

```

和前面做的一样，我们只定义了两个字段（Character_Name和Class_ID）。<br>

在编译后，创建一个数据表格（<font color=#BD63C5 size=3>Data Table</font>）并选择<font color=#BD63C5 size=3>CharacterInfo</font>作为类别，添加一个新的条目取名Character_Name为S1，你也可以取一个你喜欢的名字，但是class
ID必须填写S1（在之前我们定义了名称为S1的小兵类别）

----------

### **<font color=#191970 size=4>敌人</font> ** ###

至于敌人，我们不是单独定义个角色和单独的类别信息。我们会吧两个部分的信息简单的结合起来。作为一个敌人，通常不会处理获得经验和升级，所以我们可以省略这部分相关的数据。除此之外，敌人也不会像玩家一样消耗MP，我们也可以省略与此相关的数据。<br>

因为上面介绍的那些原因，我们的敌人数据会包含以下属性：

- <font color=#BD63C5 size=3>名称（整数）</font>
- <font color=#BD63C5 size=3>最大生命值（整数）</font>
- <font color=#BD63C5 size=3>攻击力（整数）</font>
- <font color=#BD63C5 size=3>防御力（整数）</font>
- <font color=#BD63C5 size=3>幸运值（整数）</font>

现在，你应该了解如何构造这个类的数据。<br>
先让我们看看这个结构的头文件：

- FEnemieInfo.h

``` cpp

USTRUCT( BlueprintType )
struct FEnemyInfo : public FTableRowBase
{
    GENERATED_USTRUCT_BODY()
    UPROPERTY( BlueprintReadWrite, EditAnywhere, Category = "EnemyInfo" )
    FString EnemyName;
    UPROPERTY( BlueprintReadOnly, EditAnywhere, Category = "EnemyInfo" )
    int32 MHP;
    UPROPERTY( BlueprintReadOnly, EditAnywhere, Category = "EnemyInfo" )
    int32 ATK;
    UPROPERTY( BlueprintReadOnly, EditAnywhere, Category = "EnemyInfo" )
    int32 DEF;
    UPROPERTY( BlueprintReadOnly, EditAnywhere, Category = "EnemyInfo" )
    int32 Luck;
    UPROPERTY( BlueprintReadOnly, EditAnywhere, Category = "EnemyInfo" )
    TArray<FString> Abilities;
};

```

在编译之后，创建一个新的数据表格<font color=#BD63C5 size=3>Data Table</font>）并选择<font color=#BD63C5 size=3>EnemyInfo</font>作为类别，添加一个名叫S1的新条目，新条目具有以下属性：

- <font color=#BD63C5 size=3>Enemy name:</font> Goblin
- <font color=#BD63C5 size=3>MHP:</font> 100
- <font color=#BD63C5 size=3>ATK:</font> 5
- <font color=#BD63C5 size=3>DEF:</font> 0
- <font color=#BD63C5 size=3>Luck:</font> 0
- <font color=#BD63C5 size=3>Abilities:</font> 现在为空

现在我们有了一个角色数据，一个角色类别和一个角色可以战斗的敌人。下一步，我们将会开始追踪那些角色是活动的和他们当前统计数据是什么。

----------

### **<font color=#191970 size=4>伙伴成员</font> ** ###

在我们可以追踪伙伴成员前，我们需要一种方法来追踪一个角色的状态，比如说角色还有多少HP，角色穿了什么装备。<br>

要做到这一点，我们需要新创建一个类命名为GameCharacter，像往常一样创建一个新类，但这次需要选择Object作为父类。<br>

此头文件的代码会和以下代码一样：

- GameCharacter.h

``` cpp

#include "Data/FCharacterInfo.h"
#include "Data/FCharacterClassInfo.h"
#include "GameCharacter.generated.h"

UCLASS( BlueprintType )
class RPG_API UGameCharacter : public UObject
{
    GENERATED_BODY()
public:
    FCharacterClassInfo* ClassInfo;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    FString CharacterName;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 MHP;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 MMP;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 HP;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 MP;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 ATK;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 DEF;

    UPROPERTY( EditAnywhere, BlueprintReadWrite, Category = CharacterInfo )
    int32 LUCK;

public:

    static UGameCharacter* CreateGameCharacter( FCharacterInfo* characterInfo, UObject* outer );

public:

    void BeginDestroy() override;
};

```

现在我们角色的类信息、角色的名称、角色当前的统计数据。之后我们会用<font color=#dd1144 size=3>UCLASS</font>和<font color=#dd1144 size=3>UPROPERTY</font>宏去暴露信息给蓝图。在这之后我们会添加一些在战斗系统中用到的信息。

GameCharacter.cpp的代码会像这样：

- GameCharacter.cpp

``` cpp

UGameCharacter::UGameCharacter( const class FObjectInitializer& objectInitializer )
    :Super( objectInitializer )
{
}

UGameCharacter* UGameCharacter::CreateGameCharacter( FCharacterInfo* characterInfo, UObject* outer )
{
    UGameCharacter* character = NewObject<UGameCharacter>( outer );

    // locate character classes asset 
    UDataTable* characterClasses = Cast<UDataTable>( StaticLoadObject( UDataTable::StaticClass(), NULL, TEXT( "DataTable'/Game/Data/CharacterClasses.CharacterClasses'" ) ) );
    
    if( characterClasses == NULL )
    {
        UE_LOG( LogTemp, Error, TEXT( "Character classes datatable not found!") );
    }
    else
    {
        character->CharacterName = characterInfo->Character_Name;
        FCharacterClassInfo* row = characterClasses->FindRow<FCharacterClassInfo>( *( characterInfo->Class_ID ), TEXT( "LookupCharacterClass" ) );
        character->ClassInfo = row;
        character->MHP = character->ClassInfo->StartMHP;
        character->MMP = character->ClassInfo->StartMMP;
        character->HP = character->MHP;
        character->MP = character->MMP;
        character->ATK = character->ClassInfo->StartATK;
        character->DEF = character->ClassInfo->StartDEF;
        character->LUCK = character->ClassInfo->StartLuck;
    }
    return character;
}

void UGameCharacter::BeginDestroy()
{
    Super::BeginDestroy();
}

```

<font color=#dd1144 size=3>UGameCharacter</font>类的<font color=#dd1144 size=3>CreateGameCharacter</font>方法接收一个从DataTable返回的指向<font color=#dd1144 size=3>FCharacterInfo</font>的指针和产生这个Character的对象，用于传递给<font color=#dd1144 size=3>NewObject</font>函数。然后尝试用一个路径找这个类的DataTable，接着如果结果不为空，则从DataTable中正确读取了一行数据，并被储存。接着用这些读取的数据来初始化Character的统计信息和CharacterName字段。在上面的代码，你可以看到角色DataTable的所在路径，这个路径你可以通过右键点击DataTable，然后选择Copy Reference选项，然后你就可以在你的代码中粘贴路径了。<br>

虽然现在的角色光秃秃的，但是他可以使用。接下来我们要存储这些角色到当前的伙伴列表。

----------

### **<font color=#191970 size=4>GameInstance类</font> ** ###

我们已经创建了一个GameMode（游戏模式）类，这个类看起来是我们追踪和存储伙伴成员的完美地方，是吧?<br>

然而，GameMode（游戏模式）在关卡不同关卡读取时不会保存数据，除非你把这些数据信息存储到了磁盘，每当你到了一个新的区域你将会丢失你的所有数据。<br>

下面我们来介绍一下为了解决这种问题的<font color=#dd1144 size=3>GameInstance</font>类，font color=#191970 size=3>AGameInstance</font>不像GameMode（游戏模式），不论关卡读取还是做些什么，他一直存在在整个游戏过程中。我们需要创建一个新的<font color=#dd1144 size=3>GameInstance</font>类来持续追踪和存储伙伴成员的信息。<br>

创建一个新类，这一次我们选择GameInstance作为父类（你需要在查找功能中查找那个类），将它取名为RPGGameInstance。

在这个头文件中，我们需要添加一个用来存储<font color=#dd1144 size=3>UGameCharacter</font>指针的<font color=#dd1144 size=3>TArray</font>，一个用来确定游戏已经被初始化的标志位和<font color=#dd1144 size=3>Init</font>函数：

- RPGGameInstance.h

``` cpp

UCLASS()
class RPG_API URPGGameInstance : public UGameInstance
{
    GENERATED_BODY()
    URPGGameInstance( const class FObjectInitializer& ObjectInitializer );

public:

    TArray<UGameCharacter*> PartyMembers;

protected:

    bool isInitialized;

public:

    void Init();

};

```

在游戏实例的<font color=#dd1144 size=3>Init</font>函数中，我们会添加一个单个默认的伙伴成员并且设置<font color=#dd1144 size=3>isInitialized</font>标志位为<font color=#dd1144 size=3>true</font>：

- RPGGameInstance.cpp

``` cpp

void URPGGameInstance::Init()
{
    if( this->isInitialized ) return;
    this->isInitialized = true;
    
    // locate characters asset
    UDataTable* characters = Cast<UDataTable>( StaticLoadObject( UDataTable::StaticClass(), NULL, TEXT( "DataTable'/Game/Data/Characters.Characters'" ) ) );
    
    if( characters == NULL )
    {
        UE_LOG( LogTemp, Error, TEXT( "Characters data table not found!" ) );
        return;
    }

    // locate character
    FCharacterInfo* row = characters->FindRow<FCharacterInfo>( TEXT( "S1" ), TEXT( "LookupCharacterClass" ) );

    if( row == NULL )
    {
        UE_LOG( LogTemp, Error, TEXT( "Character ID 'S1' not found!" ) );
        return;
    }
    
    // add character to party
    this->PartyMembers.Add( UGameCharacter::CreateGameCharacter( row, this ) );
}

```

在虚幻中设置<font color=#dd1144 size=3>GameInstance</font>，打开<font color=#dd1144 size=3>Edit</font>-><font color=#dd1144 size=3>Project Settings</font>跳转到<font color=#dd1144 size=3>Maps & Modes</font>，向下滑动到<font color=#dd1144 size=3>Game Instance</font>窗格，在下拉菜单中选择<font color=#dd1144 size=3>RPGGameInstance</font>。最后，我们重写GameMode（游戏模式）的<font color=#dd1144 size=3>BeginPlay</font>函数中调用这个<font color=#dd1144 size=3>Init</font>方法：

- RPGGameInstance.cpp

``` cpp

// RPGGameMode.h
virtual void BeginPlay() override;

// RPGGameMode.cpp
void ARPGGameMode::BeginPlay()
{
    Cast<URPGGameInstance>( GetGameInstance() )->Init();
}

```

现在我们有了一个活动的伙伴成员列表，是时候去实现战斗引擎了。

----------

### **<font color=#191970 size=4>回合战斗</font> ** ###

正如我们第1章“虚幻引擎RPG设计入门”所讲的，我们的战斗是回合制战斗。所有的角色先要选择一个要执行的动作。然后所有角色按照顺序依次执行动作。<br>

战斗会分为两个阶段：

- <font color=#dd1144 size=3>决策</font>，所有角色决定他们的行动方案。
- <font color=#dd1144 size=3>行动</font>，所有角色按照他们的行动方案执行。

我们需要创建一个为我们处理战斗的类，取名为<font color=#dd1144 size=3>CombatEngine</font>：

- CombatEngine.h

``` cpp

#include "RPG.h"
#include "GameCharacter.h"

enum class CombatPhase : uint8
{
    CPHASE_Decision,
    CPHASE_Action,
    CPHASE_Victory,
    CPHASE_GameOver,
};

class RPG_API CombatEngine
{

public:

    TArray<UGameCharacter*> combatantOrder;
    TArray<UGameCharacter*> playerParty;
    TArray<UGameCharacter*> enemyParty;
    CombatPhase phase;

protected:

    UGameCharacter* currentTickTarget;
    int tickTargetIndex;

public:

    CombatEngine( TArray<UGameCharacter*> playerParty, TArray<UGameCharacter*> enemyParty );
    ~CombatEngine();
    bool Tick( float DeltaSeconds );

protected:

    void SetPhase( CombatPhase phase );
    void SelectNextCharacter();
};

```

这个类很长，我要一一解释。<br>

我们的战斗引擎会在遭遇敌人时创建并且会在战斗结束时删除。<br>
一个战斗<font color=#dd1144 size=3>CombatEngine</font>的实例保存着三个<font color=#dd1144 size=3>TArrays</font>：一个是用来存储战斗顺序（所有战斗参与者的顺序列表，所有参与者会依次轮流行动），另一个是玩家列表，第三个是敌人列表。这个实例也持续追踪着<font color=#dd1144 size=3>CombatPhase</font>，战斗有两个主要的阶段：<font color=#dd1144 size=3>Decision</font>和<font color=#dd1144 size=3>Action</font>，战斗中的每一轮都从<font color=#dd1144 size=3>Decision</font>阶段开始。在这个阶段，所有的角色决定他们的行动方案。然后战斗转换为<font color=#dd1144 size=3>Action</font>阶段。在这个阶段中，所有的角色按照顺序执行之前决定的行动方案。<br>

<font color=#dd1144 size=3>GameOver</font>和<font color=#dd1144 size=3>Victory</font>会在所有敌人全部死亡或者玩家全部死亡时分别转换到这两个状态中。（这就是为什么我们要将敌人列表和玩家列表分成两个单独的列表）<br>

<font color=#dd1144 size=3>CombatEngine</font>类定义了一个<font color=#dd1144 size=3>Tick</font>方法，只要战斗没有结束，游戏模式类会每一帧都调用此方法。当战斗结束时，这个方法返回结果为<font color=#dd1144 size=3>ture</font>（没有结束返回<font color=#dd1144 size=3>false</font>），这个方法将上一帧的持续时间作为参数。<br>

还有<font color=#dd1144 size=3>currentTickTarget</font>和<font color=#dd1144 size=3>tickTargetIndex</font>，在<font color=#dd1144 size=3>Decision</font>和<font color=#dd1144 size=3>Action</font>阶段，我们会保存一个指向单个角色的指针。比如说，在<font color=#dd1144 size=3>Decision</font>阶段，在开始时指针会指向战斗顺序列表中的第一个角色。在每一帧中，都会有一个函数让这个角色做出决定。如果返回<font color=#dd1144 size=3>ture</font>表示角色已经完成了决定，如果返回<font color=#dd1144 size=3>false</font>表示角色还没有决定。如果这个函数返回了<font color=#dd1144 size=3>true</font>这个指针会指向列表中的下一个角色，然后这样一直持续到所有角色都昨晚了决定。之后战斗转到到<font color=#dd1144 size=3>Action</font>阶段。<br>

这个CPP文件很大，我们拆分成小块来看。我们先来看看构造函数和析构函数。<br>

- CombatEngine.cpp

``` cpp

CombatEngine::CombatEngine( TArray<UGameCharacter*> playerParty, TArray<UGameCharacter*> enemyParty )
{
    this->playerParty = playerParty;
    this->enemyParty = enemyParty;
    // first add all players to combat order
    for( int i = 0; i < playerParty.Num(); i++ )
    {
        this->combatantOrder.Add( playerParty[i] );
    }
    // next add all enemies to combat order
    for( int i = 0; i < enemyParty.Num(); i++ )
    {
        this->combatantOrder.Add( enemyParty[i] );
    }
    this->tickTargetIndex = 0;
    this->SetPhase( CombatPhase::CPHASE_Decision );
}

CombatEngine::~CombatEngine()
{

}

```

构造函数首先分配<font color=#dd1144 size=3>playerParty</font>和<font color=#dd1144 size=3>enemyParty</font>这两个字段，然后将所有玩家依次加入到战斗顺序列表中，再将敌人依次加入到战斗顺序列表中。最后，设置目标索引为0（即战斗顺序列表的第一个角色）和战斗阶段为<font color=#dd1144 size=3>Decision</font>阶段<br>

我们紧接着看看Tick方法：

- CombatEngine.cpp

``` cpp

bool CombatEngine::Tick( float DeltaSeconds )
{
    switch( phase )
    {
    case CombatPhase::CPHASE_Decision:
        // todo: ask current character to make decision
        // todo: if decision made
        SelectNextCharacter();
        // no next character, switch to action phase
        if( this->tickTargetIndex == -1 )
        {
            this->SetPhase( CombatPhase::CPHASE_Action );
        }
        break;
    case CombatPhase::CPHASE_Action:
        // todo: ask current character to execute decision
        // todo: when action executed
        SelectNextCharacter();
        // no next character, loop back to decision phase
        if( this->tickTargetIndex == -1 )
        {
            this->SetPhase( CombatPhase::CPHASE_Decision );
        }
        break;
    // in case of victory or combat, return true (combat is finished)
    case CombatPhase::CPHASE_GameOver:
    case CombatPhase::CPHASE_Victory:
        return true;
        break;
    }

    // check for game over
    int deadCount = 0;
    for( int i = 0; i < this->playerParty.Num(); i++ )
    {
        if( this->playerParty[ i ]->HP <= 0 ) deadCount++;
    }

    // all players have died, switch to game over phase
    if( deadCount == this->playerParty.Num() )
    {
        this->SetPhase( CombatPhase::CPHASE_GameOver );
        return false;
    }

    // check for victory
    deadCount = 0;
    for( int i = 0; i < this->enemyParty.Num(); i++ )
    {
        if( this->enemyParty[ i ]->HP <= 0 ) deadCount++;
    }

    // all enemies have died, switch to victory phase
    if( deadCount == this->enemyParty.Num() )
    {
        this->SetPhase( CombatPhase::CPHASE_Victory );
        return false;
    }

    // if execution reaches here, combat has not finished - return false
    return false;
}

```

我们先看当前值阶段是处于哪个战斗阶段，如果处于<font color=#dd1144 size=3>Decision</font>阶段我们只做了选择下一个角色这件事，如果没有角色可以选择了，则切换到<font color=#dd1144 size=3>Action</font>阶段。如果处于<font color=#dd1144 size=3>Action</font>阶段也是同样的逻辑，如果没有角色可以选择了，则循环切换回<font color=#dd1144 size=3>Decision</font>阶段<br>

之后会调用角色的方法使得他们按顺序做决定或者执行动作。（注意：选择一下一个角色这个函数只能在完成决定后或者执行动作后调用一次。）<br>

在<font color=#dd1144 size=3>GameOver</font>和<font color=#dd1144 size=3>Victory</font>阶段，<font color=#dd1144 size=3>Tick</font>返回<font color=#dd1144 size=3>true</font>意味着战斗结束了。<br>
在战斗没有结束时，函数先检查是不是所有玩家都死亡了（检查战斗是不是失败了），然后检查了所有敌人是不是死亡了（检查战斗是不是胜利了）。这个两个阶段都会返回<font color=#dd1144 size=3>true</font>表示战斗结束了。<br>

在函数的最后返回了<font color=#dd1144 size=3>false</font>来表示战斗还没有结束。<br>

接下来我们来看看<font color=#dd1144 size=3>SetPhase</font>函数：

- CombatEngine.cpp

``` cpp

void CombatEngine::SetPhase( CombatPhase phase )
{
    this->phase = phase;
    switch( phase )
    {
    case CombatPhase::CPHASE_Action:
    case CombatPhase::CPHASE_Decision:
        // set the active target to the first character in the combat order
        this->tickTargetIndex = 0;
        this->SelectNextCharacter();
        break;
    case CombatPhase::CPHASE_Victory:
        // todo: handle victory
        break;
    case CombatPhase::CPHASE_GameOver:
        // todo: handle game over
        break;
    }
}

```

这是个设置战斗阶段的函数，当战斗阶段为<font color=#dd1144 size=3>Action</font>后者<font color=#dd1144 size=3>Decision</font>时，这个函数会设置<font color=#dd1144 size=3>tickTargetIndex</font>为战斗顺序列表中的第一个。<font color=#dd1144 size=3>Victory</font>和<font color=#dd1144 size=3>GameOver</font>预留着各自的状态处理。<br>

最后我们来看<font color=#dd1144 size=3>SelectNextCharacter</font>

- CombatEngine.cpp

``` cpp

void CombatEngine::SelectNextCharacter()
{
    for( int i = this->tickTargetIndex; i < this->combatantOrder.Num(); i++ )
    {
        GameCharacter* character = this->combatantOrder[ i ];
        if( character->HP > 0 )
        {
            this->tickTargetIndex = i + 1;
            this->currentTickTarget = character;
            return;
        }
    }
    this->tickTargetIndex = -1;
    this->currentTickTarget = nullptr;
}

```

这个函数从当前<font color=#dd1144 size=3>tickTargetIndex</font>位置开始按顺序向后找到一个没有死亡的角色。如果找到一个，就将<font color=#dd1144 size=3>tickTargetIndex</font>和<font color=#dd1144 size=3>currentTickTarget</font>都设置为这个角色。如果没有找到，就将<font color=#dd1144 size=3>tickTargetIndex</font>设置为-1，<font color=#dd1144 size=3>currentTickTarget</font>设置为空指针（这意味着作战顺序列表里面已经没有存活的角色了）。<br>

现在还遗漏了一件非常重要的事情：角色还不能作出或者执行决定。<br>

让我们将这两个方法加入到<font color=#dd1144 size=3>GameCharacter</font>类中，只是作为预留的方法。<br>

首先我们添加<font color=#dd1144 size=3>testDelayTimer</font>字段，这个字段只作为测试用途。<br>

- GameCharacter.h

``` cpp
protected:
    float testDelayTimer;
```

接下来我们往类中添加几个方法。<br>

- GameCharacter.h

``` cpp
public:
    void BeginMakeDecision();
    bool MakeDecision( float DeltaSeconds );

    void BeginExecuteAction();
    bool ExecuteAction( float DeltaSeconds );
```

我们以同样的方式分离了<font color=#dd1144 size=3>Decision</font>和<font color=#dd1144 size=3>Action</font>，让他们各自拥有两个函数。第一个函数是告诉角色开始做决定或者开始执行动作，第二个函数的本质上是一直查询角色是否已经完成决定或者完成执行动作。<br>

这两个方法我们会在以后实现，现在，我们只是延迟一秒输出日志：<br>

- GameCharacter.cpp

``` cpp
void UGameCharacter::BeginMakeDecision()
{
    UE_LOG( LogTemp, Log, TEXT( "Character %s making decision" ), *this->CharacterName );
    this->testDelayTimer = 1;
}

bool UGameCharacter::MakeDecision( float DeltaSeconds )
{
    this->testDelayTimer -= DeltaSeconds;
    return this->testDelayTimer <= 0;
}
void UGameCharacter::BeginExecuteAction()
{
    UE_LOG( LogTemp, Log, TEXT( "Character %s executing action" ), *this->CharacterName );
    this->testDelayTimer = 1;
}

bool UGameCharacter::ExecuteAction( float DeltaSeconds )
{
    this->testDelayTimer -= DeltaSeconds;
    return this->testDelayTimer <= 0;
}
```

我们还要添加一个指向战斗实例的指针。因为战斗引擎已经引用了角色类，角色类在引用战斗引擎会产生循环依赖。为了避免这个问题，我们需要在<font color=#dd1144 size=3>GameCharacter.h</font>中添加前置声明。<br>

- GameCharacter.h

``` cpp
class CombatEngine;
```

然后，战斗引擎的<font color=#dd1144 size=3>include</font>语句应该放在
<font color=#dd1144 size=3>GameCharacter.cpp</font>中而不是在头文件中。<br>

接下来，我们要用战斗引擎来调用<font color=#dd1144 size=3>Decision</font>和<font color=#dd1144 size=3>Action</font>的方法，我们要先在<font color=#dd1144 size=3>CombatEngine</font>类中添加一个标志位：

- CombatEngine.h

``` cpp
bool waitingForCharacter;
```

这个标志位将用于切换。例如，在<font color=#dd1144 size=3>BeginMakeDecision</font>和<font color=#dd1144 size=3>MakeDecision</font>之前切换。 

接下来，我们要更新<font color=#dd1144 size=3>Tick</font>方法中的<font color=#dd1144 size=3>Decision</font>和<font color=#dd1144 size=3>Action</font>阶段。我们先来更新一下前面的<font color=#dd1144 size=3>Decision</font>部分。

- CombatEngine.cpp

``` cpp
{
    if( !this->waitingForCharacter )
    {
        this->currentTickTarget->BeginMakeDecision();
        this->waitingForCharacter = true;
    }

    bool decisionMade = this->currentTickTarget->MakeDecision( DeltaSeconds );
    if( decisionMade )
    {
        SelectNextCharacter();
        // no next character, switch to action phase
        if( this->tickTargetIndex == -1 )
        {
            this->SetPhase( CombatPhase::CPHASE_Action );
        }
    }
} 
break;
```

如果<font color=#dd1144 size=3>waitingForCharacter</font>是<font color=#dd1144 size=3>false</font>，它会调用<font color=#dd1144 size=3>BeginMakeDecision</font>方法并且设置<font color=#dd1144 size=3>waitingForCharacter</font>为<font color=#dd1144 size=3>true</font>。<br>

注意整个括号括起来的<font color=#dd1144 size=3>case</font>语句，如果你不加这个括号，<font color=#dd1144 size=3>case</font>语句会在编译时报<font color=#dd1144 size=3>decisionMade</font>初始化被跳过的错误。<br>

接着调用了<font color=#dd1144 size=3>MakeDecision</font>方法并传递了一帧的时间作为参数。如果这个方法返回<font color=#dd1144 size=3>true</font>，将会选择下一个角色。返回<font color=#dd1144 size=3>false</font>就切换到<font color=#dd1144 size=3>Action</font>阶段。<br>

<font color=#dd1144 size=3>Action</font>阶段和上面的代码几乎相同：

- CombatEngine.cpp

``` cpp
{
    if( !this->waitingForCharacter )
    {
        this->currentTickTarget->BeginExecuteAction();
        this->waitingForCharacter = true;
    }
    bool actionFinished = this->currentTickTarget->ExecuteAction( DeltaSeconds );
    if( actionFinished )
    {
        SelectNextCharacter();
        // no next character, switch to action phase
        if( this->tickTargetIndex == -1 )
        {
            this->SetPhase( CombatPhase::CPHASE_Decision );
        }
    }
}
break;
```

接着我们要更新一下<font color=#dd1144 size=3>SelectNextCharacter</font>方法，需要在这个方法中将<font color=#dd1144 size=3>waitingForCharacter</font>设置回<font color=#dd1144 size=3>false</font>

- CombatEngine.cpp

``` cpp
void CombatEngine::SelectNextCharacter()
{
    this->waitingForCharacter = false;
    // ...(原先代码)
}
```

最后，我们还要完善一些细节：我们的战斗引擎需要设置所有的角色的<font color=#dd1144 size=3>CombatInstance</font>的指针指向自己，我们需要在构造函数里做这些。然后我们还需要在析构函数中清空这些指针和敌人的指针：

- CombatEngine.cpp

``` cpp
CombatEngine::CombatEngine( TArray<UGameCharacter*> playerParty, TArray<UGameCharacter*> enemyParty )
{
    // ...
    for( int i = 0; i < this->combatantOrder.Num(); i++ )
    {
        this->combatantOrder[i]->combatInstance = this;
    }
    this->tickTargetIndex = 0;
    this->SetPhase( CombatPhase::CPHASE_Decision );
}

CombatEngine::~CombatEngine()
{
    // free enemies
    for( int i = 0; i < this->enemyParty.Num(); i++ )
    {
        this->enemyParty[i] = nullptr;
    }

    for( int i = 0; i < this->combatantOrder.Num(); i++ )
    {
        this->combatantOrder[i]->combatInstance = nullptr;
    }
}
```

现在战斗引擎到功能已经完整了，我们还需要把它挂钩到游戏中。我们要在游戏模式中去触发战斗和更新战斗。<br>

所以在我们的游戏模式类中，我们需要添加个指针去指向当前战斗。然后重写游戏模式类的<font color=#dd1144 size=3>Tick</font>方法。此外还的保存一个追踪角色的列表（修饰符要用<font color=#dd1144 size=3>UPROPERTY</font>，这样敌人就可以被垃圾回收了）：

- RPGGameMode.h

``` cpp
UCLASS()
class RPG_API ARPGGameMode : public AGameMode
{
    GENERATED_BODY()

    ARPGGameMode( const class FObjectInitializer& ObjectInitializer );
    virtual void Tick( float DeltaTime ) override;

public:
    
    CombatEngine* currentCombatInstance;
    TArray<UGameCharacter*> enemyParty;
};
```

接着在cpp文件中我们来实现它的<font color=#dd1144 size=3>Tick</font>方法：

- RPGGameMode.cpp

``` cpp
void ARPGGameMode::Tick( float DeltaTime )
{
    if( this->currentCombatInstance != nullptr )
    {
        bool combatOver = this->currentCombatInstance->Tick( DeltaTime );
        if( combatOver )
        {
            if( this->currentCombatInstance->phase == CombatPhase::CPHASE_GameOver )
            {
                UE_LOG( LogTemp, Log, TEXT( "Player loses combat, game over" ) );
            }
            else if( this->currentCombatInstance->phase == CombatPhase::CPHASE_Victory )
            {
                UE_LOG( LogTemp, Log, TEXT( "Player wins combat" ) );
            }
            // enable player actor
            UGameplayStatics::GetPlayerController( GetWorld(), 0 )->SetActorTickEnabled( true );

            delete( this->currentCombatInstance );
            this->currentCombatInstance = nullptr;
            this->enemyParty.Empty();
        }
    }
}
```

我们现在只检查是否有当前战斗实例。如果有，则调用战斗实例的<font color=#dd1144 size=3>Tick</font>方法。如果返回<font color=#dd1144 size=3>true</font>，则检查状态是胜利了还是失败了。（现在我们只是输出了日志在控制台）。然后，删除了了战斗实例，设置当前战斗实例为空，然后清空了敌方的角色列表（因为列表有<font color=#dd1144 size=3>UPROPERTY</font>修饰符，会使列表内的敌人自动被垃圾回收），在这我们还启用了玩家的<font color=#dd1144 size=3>Tick</font>方法。（我们会在战斗开始时禁用玩家的<font color=#dd1144 size=3>Tick</font>方法，所以玩家会在战斗时冻结在原地）<br>

我们也已经准备好遭遇敌人了，但是现在没有敌人和我们战斗。<br>

我们已经定义了敌人信息的表，但是我们的<font color=#dd1144 size=3>GameCharacter</font>类还不支持用<font color=#dd1144 size=3>EnemyInfo</font>来初始化敌人（前面我们只实现了初始化玩家）。

为了解决这个问题，我们需要在<font color=#dd1144 size=3>GameCharacter</font>类中创建一个工厂方法（确定你也在头部添加了<font color=#dd1144 size=3>EnemyInfo</font>类的<font color=#dd1144 size=3>include</font>语句）：

- GameCharacter.h

``` cpp
static UGameCharacter* CreateGameCharacter( FEnemyInfo* enemyInfo, UObject* outer );
```

我们也得实现这个重载方法：

- GameCharacter.cpp

``` cpp
UGameCharacter* UGameCharacter::CreateGameCharacter( FEnemyInfo* enemyInfo, UObject* outer )
{
    UGameCharacter* character = NewObject<UGameCharacter>( outer );
    character->CharacterName = enemyInfo->EnemyName;
    character->ClassInfo = nullptr;
    character->MHP = enemyInfo->MHP;
    character->MMP = 0;
    character->HP = enemyInfo->MHP;
    character->MP = 0;
    character->ATK = enemyInfo->ATK;
    character->DEF = enemyInfo->DEF;
    character->LUCK = enemyInfo->Luck;
    return character;
}
```

这是一个比较简单的实现，简单分配了名称，<font color=#dd1144 size=3>ClassInfo</font>为空（因为敌人并没有与他们关联的类）和其他的统计数据（<font color=#dd1144 size=3>MMP</font>和<font color=#dd1144 size=3>MP</font>都设置为0，因为敌人不用消耗<font color=#dd1144 size=3>MP</font>）。<br>

为了测试我们的战斗系统，我们在<font color=#dd1144 size=3>RPGGameMode.h</font>中创建了一个函数，这个函数可以在虚幻控制台调用。

- RPGGameMode.h

``` cpp
UFUNCTION(exec)
void TestCombat();
```

<font color=#dd1144 size=3>UFUNCTION(exec)</font>宏可以让这个函数可以在虚幻控制台中使用命令调用。<br>

<font color=#dd1144 size=3>RPGGameMode.cpp</font>中此方法的实现如下：

- RPGGameMode.cpp

``` cpp
void ARPGGameMode::TestCombat()
{
    // locate enemies asset
    UDataTable* enemyTable = Cast<UDataTable>( StaticLoadObject
        ( UDataTable::StaticClass()
        , NULL
        , TEXT( "DataTable'/Game/Data/Enemies.Enemies'" ) 
        ) );

    if( enemyTable == NULL )
    {
        UE_LOG( LogTemp, Error, TEXT( "Enemies data table not found!" ) );
        return;
    }

    // locate enemy
    FEnemyInfo* row = enemyTable->FindRow<FEnemyInfo>( TEXT( "S1" ), TEXT( "LookupEnemyInfo" ) );

    if( row == NULL )
    {
        UE_LOG( LogTemp, Error, TEXT( "Enemy ID 'S1' not found!" ) );
        return;
    }

    // disable player actor
    UGameplayStatics::GetPlayerController( GetWorld(), 0 )->SetActorTickEnabled( false );
    
    // add character to enemy party
    UGameCharacter* enemy = UGameCharacter::CreateGameCharacter( row, this );
    this->enemyParty.Add( enemy );
    
    URPGGameInstance* gameInstance = Cast<URPGGameInstance>( GetGameInstance() );
    
    this->currentCombatInstance = new CombatEngine( gameInstance->PartyMembers, this->enemyParty );

    UE_LOG( LogTemp, Log, TEXT( "Combat started" ) );
}
```

在这我们创建了一个敌人的<font color=#dd1144 size=3>DataTable</font>，并选择了ID为S1的敌人创造了一个<font color=#dd1144 size=3>GameCharacter</font>，紧接着创造了一个敌人的列表来添加这些敌人。然后创建了一个<font color=#dd1144 size=3>CombatEngine</font>的实例传递给了玩家方，敌人的列表传给了敌方。我们还必须在战斗开始的时候禁用<font color=#dd1144 size=3>Tick</font>方法，来停止对玩家的更新。<br>

最后，我们必须测试一下战斗引擎，开始游戏后按键盘的<font color=#dd1144 size=3>(~)</font>键来调出控制台命令行窗口，输入<font color=#dd1144 size=3>TestCombat</font>然后按<font color=#dd1144 size=3>Enter</font>键。<br>

在输出窗口，你可以看到一些和下面信息类似的信息：

>LogTemp: Combat started<br>
>LogTemp: Character Kumo making decision<br>
>LogTemp: Character Goblin making decision<br>
>LogTemp: Character Kumo executing action<br>
>LogTemp: Character Goblin executing action<br>
>LogTemp: Character Kumo making decision<br>
>LogTemp: Character Goblin making decision<br>
>LogTemp: Character Kumo executing action<br>
>LogTemp: Character Goblin executing action<br>
>LogTemp: Character Kumo making decision<br>
>LogTemp: Character Goblin making decision<br>
>LogTemp: Character Kumo executing action<br>
>LogTemp: Character Goblin executing action<br>
>LogTemp: Character Kumo making decision<br>

首先这些信息表明战斗引擎正在像预期一样的运行。所有的角色都做出一个决定，然后去执行决定。接着他们又会做出决定，然后继续去执行决定，然后一直持续下去。因为没有人做任何事情（更不会造成任何伤害），所以战斗会一直持续下去。<br>

现在还有两个问题围绕着我们：第一，就是前面提到的问题，没有一个角色真正的做任何事情。此外，玩家角色需要一个与敌人不同的方式来作出决定（玩家角色需要一个UI去选择动作来作出决定，相反敌人角色需要自动的作出决定）<br>

我们会在解决决策问题之前先解决第一个问题。

----------

### **<font color=#191970 size=4>执行动作</font> ** ###

为了能让角色执行动作，我们要把所有的战斗动作归为一个通用的接口。我们现在已经有了映射这些接口的好地方。那就是角色的<font color=#dd1144 size=3>BeginExecuteAction</font>和<font color=#dd1144 size=3>ExecuteAction</font>这两个方法。

让我们像下面一样创建一个新的接口<font color=#dd1144 size=3>ICombatAction</font>：

- CombatAction.h

``` cpp
#pragma once
#include "GameCharacter.h"

class UGameCharacter;

class ICombatAction
{ 
public:
    
    virtual void BeginExecuteAction( UGameCharacter* character ) = 0;
    virtual bool ExecuteAction( float DeltaSeconds ) = 0;
};
```

<font color=#dd1144 size=3>BeginExecuteAction</font>接收一个指向正在执行动作的角色的指针<br>
<font color=#dd1144 size=3>ExecuteAction</font>像之前一样，接收上一帧的时间作为参数<br>

接着我们创建一个新的动作类来实现这些接口。作为测试，我们在新类<font color=#dd1144 size=3>TestCombatAction</font>中复制前面角色已经做的功能（也就是什么都没有，打印些日志）：

头文件的代码会是下面这样：

- TestCombatAction.h

``` cpp
#pragma once

#include "ICombatAction.h"

class TestCombatAction : public ICombatAction
{ 
protected:

    float delayTimer;
public:

    virtual void BeginExecuteAction( UGameCharacter* character ) override;
    virtual bool ExecuteAction( float DeltaSeconds ) override;
};
```

cpp代码会是下面这样：

- TestCombatAction.cpp

``` cpp
#include "RPG.h"
#include "TestCombatAction.h"

void TestCombatAction::BeginExecuteAction( UGameCharacter* character )
{
    UE_LOG( LogTemp, Log, TEXT( "%s does nothing" ), *character->CharacterName );
    this->delayTimer = 1.0f;
}

bool TestCombatAction::ExecuteAction( float DeltaSeconds )
{
    this->delayTimer -= DeltaSeconds;
    return this->delayTimer <= 0.0f;
}
```

接着，我们要修改角色类能够存储和执行这些动作。<br>

首先，将角色类中测试用的<font color=#dd1144 size=3>delayTimer</font>字段替换成一个战斗动作的指针。然后在我们需要在创建决策系统时公开这个字段。

- GameCharacter.h

``` cpp
public:
    ICombatAction* combatAction;
```

接着我们需要在决策函数中分配一个战斗动作，在执行函数中执行这个动作：

- GameCharacter.cpp

``` cpp
void UGameCharacter::BeginMakeDecision()
{
    UE_LOG( LogTemp, Log, TEXT( "Character %s making decision" ), *( this->CharacterName ) );
    this->combatAction = new TestCombatAction();
}

bool UGameCharacter::MakeDecision( float DeltaSeconds )
{
    return true;
}

void UGameCharacter::BeginExecuteAction()
{
    this->combatAction->BeginExecuteAction( this );
}

bool UGameCharacter::ExecuteAction( float DeltaSeconds )
{
    bool finishedAction = this->combatAction->ExecuteAction( DeltaSeconds );
    if( finishedAction )
    {
        delete( this->combatAction );
        return true;
    }
    return false;
}
```

<font color=#dd1144 size=3>BeginMakeDecision</font>现在分配了一个<font color=#dd1144 size=3>TestCombatAction</font>的实例，<font color=#dd1144 size=3>MakeDecision</font>只是返回了<font color=#dd1144 size=3>true</font>，<font color=#dd1144 size=3>BeginExecuteAction</font>方法，用存储的动作调用了相同名称的方法，并且传递了这个角色的指针作为参数。最后，<font color=#dd1144 size=3>ExecuteAction</font>函数，也使用存储的动作调用了同名的方法并且得到了个结果，如果结果是<font color=#dd1144 size=3>true</font>则删除指针并且返回<font color=#dd1144 size=3>true</font>，相反则返回<font color=#dd1144 size=3>false</font>。<br>

让我们再次测试一下新的代码，你会发现在输出窗口会输出同样的日志信息，但现在它的作用是说明做什么而不是怎么做。<br>

现在我们已经有个方法来存储和执行动作了，接着我们要来实现我们的角色决策系统了。

----------

### **<font color=#191970 size=4>决策</font> ** ###

我们会像之前做执行动作一样，为决策系统重新创建一个接口，类似于<font color=#dd1144 size=3>BeginMakeDecision</font>/<font color=#dd1144 size=3>MakeDecision</font>这样的模式。<font color=#dd1144 size=3>IDecisionMaker</font>会像下面这样：

- DecisionMaker.h

``` cpp
#pragma once

#include "GameCharacter.h"

class UGameCharacter;

class IDecisionMaker
{ 
public:
    
    virtual void BeginMakeDecision( UGameCharacter* character ) = 0;
    virtual bool MakeDecision( float DeltaSeconds ) = 0;
};
```

然后，我们要创建<font color=#dd1144 size=3>TestDecisionMaker</font>来实现接口：

- TestDecisionMaker.h

``` cpp
// TestDecisionMaker.h
#pragma once

#include "IDecisionMaker.h"

class RPG_API TestDecisionMaker : public IDecisionMaker
{
public:
    
    virtual void BeginMakeDecision( UGameCharacter* character ) override;
    virtual bool MakeDecision( float DeltaSeconds ) override;
};
```

- TestDecisionMaker.cpp

``` cpp

// TestDecisionMaker.CPP

#include "RPG.h"
#include "TestDecisionMaker.h"
#include "../Actions/TestCombatAction.h"

void TestDecisionMaker::BeginMakeDecision( UGameCharacter* character )
{
    character->combatAction = new TestCombatAction();
}

bool TestDecisionMaker::MakeDecision( float DeltaSeconds )
{
    return true;
}
```

接着我们要往角色类里面添加一个指向<font color=#dd1144 size=3>IDecisionMaker</font>的指针，并且修改<font color=#dd1144 size=3>BeginMakeDecision</font>/<font color=#dd1144 size=3>MakeDecision</font>方法来使用决策类。

- GameCharacter.h

``` cpp
// GameCharacter.h
public:
    IDecisionMaker* decisionMaker;
```

- GameCharacter.cpp

``` cpp
// GameCharacter.cpp
void UGameCharacter::BeginDestroy()
{
    Super::BeginDestroy();
    delete( this->decisionMaker );
}

void UGameCharacter::BeginMakeDecision()
{
    this->decisionMaker->BeginMakeDecision( this );
}

bool UGameCharacter::MakeDecision( float DeltaSeconds )
{
    return this->decisionMaker->MakeDecision( DeltaSeconds );
}
```

现在我们在<font color=#dd1144 size=3>BeginDestroy</font>函数中删除决策类对象吗，决策类对象会在角色创建时分配，并且他们被摧毁前删除这个对象。<br>

最后一步当然是在构造函数中分配决策类对象，在所有角色类的构造函数中添加以下代码：

- GameCharacter.cpp

``` cpp
// GameCharacter.cpp
    this->decisionMaker = new TestDecisionMaker();
```

重新运行游戏，再次测试战斗，你可以在输出窗口看到完全一样的输出。然而，有个很大的区别，现在可以实现不同的角色被分配不同的决策，并且选择决策可以方便的去分配战斗动作去执行。例如，现在我们很容易去测试一个对目标造成伤害的动作。但是在这之前，我们先对<font color=#dd1144 size=3>GameCharacter</font>类做一些小小的改动。

----------

### **<font color=#191970 size=4>目标选择</font> ** ###

我们需要在<font color=#dd1144 size=3>GameCharacter</font>类中添加个字段来标识这个是个角色、还是玩家、还是敌人。另外我们还要添加一个<font color=#dd1144 size=3>SelectTarget</font>方法用来从当前战斗实例中的玩家列表或者敌人列表中，选择第一个存活的角色，怎么选择是取决去这个角色是玩家还是敌人。<br>

我们先在<font color=#dd1144 size=3>GameCharacter.h</font>中添加一个<font color=#dd1144 size=3>isPlayer</font>字段：

- GameCharacter.h

``` cpp
    bool isPlayer;
```

紧接着我们还要添加一个<font color=#dd1144 size=3>SelectTarget</font>方法

- GameCharacter.h

``` cpp
    UGameCharacter* SelectTarget();
```

在<font color=#dd1144 size=3>GameCharacter.cpp</font>文件中我们需要在创建角色的函数中给这个字段赋值。（这是很简单的，因为我们的玩家和敌人拥有独立的创建函数）

- GameCharacter.cpp

``` cpp
UGameCharacter* CreateGameCharacter( FCharacterInfo* characterInfo, UObject* outer )
{
    //...(原有代码)
    character->isPlayer = true;
    return character;
}

UGameCharacter* CreateGameCharacter( FEnemyInfo* enemyInfo, UObject* outer )
{
    // ...(原有代码)
    character->isPlayer = false;
    return character;
}
```

接着我们需要定义<font color=#dd1144 size=3>SelectTarget</font>方法：

- GameCharacter.cpp

``` cpp
UGameCharacter* UGameCharacter::SelectTarget()
{
    UGameCharacter* target = nullptr;
    
    TArray<UGameCharacter*> targetList = this->combatInstance->enemyParty;
    if( !this->isPlayer )
    {
        targetList = this->combatInstance->playerParty;
    }

    for( int i = 0; i < targetList.Num(); i++ )
    {
        if( targetList[ i ]->HP > 0 )
        {
            target = targetList[i];
            break;
        }
    }

    if( target->HP <= 0 )
    {
        return nullptr;
    }
    return target;
}
```

首先计算出我们需要在哪个列表（玩家列表和敌人列表）中选择我们的目标，然后遍历列表去寻找一个没有死亡的目标。如果没有找到，这个函数返回一个空指针。

----------

### **<font color=#191970 size=4>造成伤害</font> ** ###

现在有了一个简单选择目标的方式，让我们修改<font color=#dd1144 size=3>TestCombatAction</font>类，使这个类用这个简单的方式来选择目标，然后我们尝试对目标造成伤害。<br>

我们先添加两个字段来维护对角色和目标的引用并且让我们的构造函数接收一个<font color=#dd1144 size=3>GameCharacter</font>目标作为参数：

- TestCombatAction.h

``` cpp
protected:
    UGameCharacter* character;
    UGameCharacter* target;
public:
    TestCombatAction( UGameCharacter* target );
```

下面是实现的代码：

- TestCombatAction.cpp

``` cpp
TestCombatAction::TestCombatAction( UGameCharacter* target )
{
    this->target = target;
}

void TestCombatAction::BeginExecuteAction( UGameCharacter* character )
{
    this->character = character;
    
    // target is dead, select another target
    if( this->target->HP <= 0 )
    {
        this->target = this->character->SelectTarget();
    }
    
    // no target, just return
    if( this->target == nullptr )
    {
        return;
    }

    UE_LOG( LogTemp, Log, TEXT( "%s attacks %s" ), *character->CharacterName, *target->CharacterName );
    
    target->HP -= 10;
    this->delayTimer = 1.0f;
}
```

首先在构造函数中对<font color=#dd1144 size=3>target</font>进行赋值。然后在<font color=#dd1144 size=3>BeginExecuteAction</font>方法中，先对<font color=#dd1144 size=3>character</font>进行赋值，紧接着检查目标是否存活。如果目标已经死亡，就会调用我们刚刚创建的<font color=#dd1144 size=3>SelectTarget</font>方法来获取一个新目标。如果获得的新目标也为空，这意味着函数返回为空，也就是说没有可用的目标了。相反如果找到了新目标，将会输出一条格式为<font color=#dd1144 size=3>[character] attacks [target]</font>的日志，最后扣除目标一部分HP，然后设置<font color=#dd1144 size=3>delayTimer</font><br>

下一步就是修改我们的<font color=#dd1144 size=3>TestDecisionMaker</font>去选择一个目标并且将这个目标传给<font color=#dd1144 size=3>TestCombatAction</font>的构造函数，这是一个比较简单的修改：

- TestDecisionMaker.cpp

``` cpp
void TestDecisionMaker::BeginMakeDecision( UGameCharacter* character )
{
    // pick a target
    UGameCharacter* target = character->SelectTarget();
    character->combatAction = new TestCombatAction( target );
}
```

现在你可以运行游戏，测试一次遭遇战斗，你会在输出窗口看到类似下面的信息：

>LogTemp: Combat started
>LogTemp: Kumo attacks Goblin
>LogTemp: Goblin attacks Kumo
>LogTemp: Kumo attacks Goblin
>LogTemp: Player wins combat

现在，我们有了一个两方可以互相攻击，并且有一方会获胜的战斗系统<br>

下一步，我们要将这些与用户界面连接

----------

### **<font color=#191970 size=4>用UMG制作战斗UI</font> ** ###

首先，我们需要设置我们的工程以确保正确的引入了<font color=#dd1144 size=3>UMG</font>和<font color=#dd1144 size=3>Slate</font>相关类。<br>

打开<font color=#dd1144 size=3>RPG.Build.cs</font>（也就是<font color=#dd1144 size=3>[ProjectName].Build.cs</font>）并且找到下面这行并修改成这样：

- [ProjectName].Build.cs

``` cs
PublicDependencyModuleNames.AddRange( 
    new string[] { 
        "Core", 
        "CoreUObject",
        "Engine", 
        "InputCore", 
        "UMG", 
        "Slate", 
        "SlateCore" 
    } 
);
```

这句语句的意思是，将<font color=#dd1144 size=3>UMG</font>、<font color=#dd1144 size=3>Slate</font>、<font color=#dd1144 size=3>SlateCore</font>添加到现有字符串数组。<br>

接着，打开<font color=#dd1144 size=3>RPG.h</font>然后加入下面这几行代码：

- RPG.h

``` cpp
#include "Runtime/UMG/Public/UMG.h"
#include "Runtime/UMG/Public/UMGStyle.h"
#include "Runtime/UMG/Public/Slate/SObjectWidget.h"
#include "Runtime/UMG/Public/IUMGModule.h"
#include "Runtime/UMG/Public/Blueprint/UserWidget.h"
```

现在编译这个工程，这会需要一点时间。<br>

接着，我们创建一个战斗UI的基类。基本上，我们使用这个基类通过定义<font color=#dd1144 size=3>Blueprint-implementable</font>在函数头部来允许C++游戏代码与蓝图UMG代码通信，这个函数可以在蓝图里实现并用C++调用<br>

创建一个新类命名为<font color=#dd1144 size=3>CombatUIWidget</font>并且选择<font color=#dd1144 size=3>UserWidget</font>作为父类：

- CombatUIWidget.h

``` cpp

#include "GameCharacter.h"
#include "Blueprint/UserWidget.h"
#include "CombatUIWidget.generated.h"

UCLASS()
class RPG_API UCombatUIWidget : public UUserWidget
{
    GENERATED_BODY()

public:

    UFUNCTION( BlueprintImplementableEvent, Category = "Combat UI" )
    void AddPlayerCharacterPanel( UGameCharacter* target );
   
    UFUNCTION( BlueprintImplementableEvent, Category = "Combat UI" )
    void AddEnemyCharacterPanel( UGameCharacter* target );
};
```

大多数情况下，我们只会定义几个个函数。<font color=#dd1144 size=3>AddPlayerCharacterPanel</font>和<font color=#dd1144 size=3>AddEnemyCharacterPanel</font>函数接收一个指向角色的指针和生成一个该角色的窗口控件。（用来显示当前角色的统计数据）。<br>

然后我们编译下代码，完成后返回编辑器，创建一个新的<font color=#dd1144 size=3>Widget Blueprint</font>命名为<font color=#dd1144 size=3>CombatUI</font>，创建完成后打开它。选择<font color=#dd1144 size=3>File</font>-><font color=#dd1144 size=3>Reparent Blueprint</font>并且选择<font color=#dd1144 size=3>CombatUIWidget</font>作为父类。<br>

在<font color=#dd1144 size=3>Designer</font>界面中，创建两个<font color=#dd1144 size=3>Horizontal Box</font>窗口控件并分别命名为<font color=#dd1144 size=3>enemyPartyStatus</font>和<font color=#dd1144 size=3>playerPartyStatus</font>，他们将会分别拥有很多玩家和敌人的子控件，去显示他们的角色统计数据。对于他们，一定要启用<font color=#dd1144 size=3>Is Variable</font>选项框，他们就对蓝图来说是可用的变量，保存并编译蓝图。<br>

然后，我们要为玩家和敌人创建显示角色统计数据的控件，我们先要创建一个需要被其他空间继承的基础控件。<br>

创建一个新的<font color=#dd1144 size=3>Widget Blueprint</font>命名为<font color=#dd1144 size=3>BaseCharacterCombatPanel</font>，在这个蓝图中，添加一个新变量<font color=#dd1144 size=3>CharacterTarget</font>并选择<font color=#dd1144 size=3>Game Character</font>作为<font color=#dd1144 size=3>Object Reference</font>类别。<br>

然后，我们要为玩家和敌人做各自的控件。<br>

创建一个新的<font color=#dd1144 size=3>Widget Blueprint</font>命名为<font color=#dd1144 size=3>PlayerCharacterCombatPanel</font>，设置新蓝图的父类为<font color=#dd1144 size=3>BaseCharacterCombatPanel</font>。

在<font color=#dd1144 size=3>Designer</font>界面中，添加三个<font color=#dd1144 size=3>Text Block</font>控件，一个为角色名称，另一个为角色HP，第三个为角色MP。我们通过在<font color=#dd1144 size=3>Details</font>面板选择控件并且点击<font color=#dd1144 size=3>Bind</font>，弹出的旁边的文本，来创建一个绑定：

![Create Binding](http://img.blog.csdn.net/20161117210304639)

这个操作将创建一个新的蓝图函数来负责生成文本。<br>

例如，想要绑定<font color=#dd1144 size=3>HP</font>文本，你需要下列步骤：

1. 拖拽<font color=#dd1144 size=3>Character Target</font>变量到视图中，并且选择<font color=#dd1144 size=3>Get</font><br>
2. 拖拽这个节点的引脚并且在<font color=#dd1144 size=3>Variables</font>-><font color=#dd1144 size=3>Character Info</font>下选择<font color=#dd1144 size=3>Get HP</font>。<br>
3. 创建一个新的<font color=#dd1144 size=3>Format Text</font>节点，设置<font color=#dd1144 size=3>Format</font>字段为<font color=#dd1144 size=3>HP: {HP}</font>，然后连接<font color=#dd1144 size=3>Get HP</font>的输出到<font color=#dd1144 size=3>Format Text</font>节点的<font color=#dd1144 size=3>HP</font>字段的输入。<br>
4. 连接<font color=#dd1144 size=3>Format Text</font>节点的输出到<font color=#dd1144 size=3>Return</font>节点的<font color=#dd1144 size=3>Return Value</font><br>

你可以重复以上步骤来创建角色名称和MP的文本。<br>

在你完成了<font color=#dd1144 size=3>PlayerCharacterCombatPanel</font>之后，你可以用同样的步骤来创建<font color=#dd1144 size=3>EnemyCharacterCombatPanel</font>，除了不要创建MP的文本块（就像前面讲的，敌人并不用消耗MP）。<br>

最终MP的展现视图的画面会像下面这样：

![MP文本快](http://img.blog.csdn.net/20161118102045697)

现在我们有了玩家和敌人的控件，让我们在<font color=#dd1144 size=3>CombatUI</font>蓝图中实现<font color=#dd1144 size=3>AddPlayerCharacterPanel</font>和<font color=#dd1144 size=3>AddEnemyCharacterPanel</font>函数。<br>

我们要先创建一个帮助函数来创建角色统计数据控件，函数命名为<font color=#dd1144 size=3>SpawnCharacterWidget</font>并且加入下列输入参数：

- <font color=#dd1144 size=3>Target Character</font>（游戏角色引用类型）
- <font color=#dd1144 size=3>Target Panel</font>（面板控件引用类型）
- <font color=#dd1144 size=3>Class</font>（基础战斗角色面板类）

这个函数需要执行下列步骤：

1. 为传入的<font color=#dd1144 size=3>Class</font>创建一个新控件
2. 转换这个新控件为<font color=#dd1144 size=3>BaseCharacterCombatPanel</font>类型
3. 设置<font color=#dd1144 size=3>Character Target</font>为输入的<font color=#dd1144 size=3>TargetCharacter</font>
4. 把这个新控件作为<font color=#dd1144 size=3>TargetPanel</font>的子控件。

蓝图最终会像下面这样：

![SpawnCharacterWidget](http://img.blog.csdn.net/20161118105946686)

然后，在<font color=#dd1144 size=3>CombatUI</font>蓝图的事件视图中，右键点击添加<font color=#dd1144 size=3>EventAddPlayerCharacterPanel</font>和<font color=#dd1144 size=3>EventAddEnemyCharacterPanel</font>事件，将他们各自挂钩一个<font color=#dd1144 size=3>SpawnCharacterWidget</font>节点，将<font color=#dd1144 size=3>Target</font>输出连接到<font color=#dd1144 size=3>Target
Character</font>输入并且将合适的变量连接到<font color=#dd1144 size=3>Target Panel</font>的输入，如下：

![CombatUI Events](http://img.blog.csdn.net/20161118111348248)

最后在我们的游戏模式中的战斗开始的地方生成这个UI，并且在战斗结束的时候摧毁这个UI，在<font color=#dd1144 size=3>RPGGameMode</font>的头文件中，添加一个<font color=#dd1144 size=3>UCombatUIWidget</font>指针和一个创建这个战斗UI的类（我们可以选择一个蓝图控件来继承我们的<font color=#dd1144 size=3>CombatUIWidget</font>类）：

- RPGGameMode.h

``` cpp
UPROPERTY()
UCombatUIWidget* CombatUIInstance;

UPROPERTY( EditDefaultsOnly, BlueprintReadOnly, Category = "UI" )
TSubclassOf<class UCombatUIWidget> CombatUIClass;
```

在我们的<font color=#dd1144 size=3>TestCombat</font>函数中，我们如下创建我们的空间实例：

- RPGGameMode.cpp

``` cpp

this->CombatUIInstance = CreateWidget<UCombatUIWidget>( GetGameInstance(),
this->CombatUIClass );
this->CombatUIInstance->AddToViewport();

for( int i = 0; i < gameInstance->PartyMembers.Num(); i++ )
{
    this->CombatUIInstance->AddPlayerCharacterPanel( gameInstance->PartyMembers[i] );
}

for( int i = 0; i < this->enemyParty.Num(); i++ )
{
    this->CombatUIInstance->AddEnemyCharacterPanel( this->enemyParty[i] );
}

```

上面的代码创建了窗口，然后添加到视图，接着分别为玩家和敌人调用他们的<font color=#dd1144 size=3>AddPlayerCharacterPanel</font>和<font color=#dd1144 size=3>AddEnemyCharacterPanel</font>函数。

在战斗结束时，我们需要从视图中移除窗口，并且设置引用为空，之后他们会被垃圾回收：

- RPGGameMode.cpp

``` cpp
this->CombatUIInstance->RemoveFromViewport();
this->CombatUIInstance = nullptr;
```

现在，如果你运行游戏，你可以看见哥布林和玩家的统计数据，他们的HP都会持续的减少直到哥布林的血量为0。然后界面消失了（因为战斗结束了）。<br>

下一步，我们要用玩家在UI上选择动作来代替自动决策。

----------

### **<font color=#191970 size=4>用UMG制作战斗UI</font> ** ###



（未完待更新）
