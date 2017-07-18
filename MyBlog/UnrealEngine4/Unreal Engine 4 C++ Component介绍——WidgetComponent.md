[TOC]

# **Unreal Engine 4 C++ Component介绍——WidgetComponent**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>WidgetComponent简介</font> ** ##

<font color=#dd1144 size=3>WidgetComponent</font>是用来渲染UI的一种组件，可以被添加到<font color=#dd1144 size=3>Actor</font>、<font color=#dd1144 size=3>Pawn</font>、<font color=#dd1144 size=3>Character</font>上。在游戏中可以用来显示角色的头顶信息以及游戏场景中的公告板。

<font color=#dd1144 size=3>WidgetComponent</font>可以渲染下列类型：

- <font color=#dd1144 size=3>UMG</font>的控件蓝图
- 继承<font color=#dd1144 size=3>UUserWidget</font>的控件
- 继承<font color=#dd1144 size=3>SWidget</font>的Slate控件

----------

## **<font color=#191970 size=5>添加UMG依赖</font> ** ##

首先，在你的工程中找到工程文件，找到以下行并添加UMG：

- <ProjectName>.Build.cs

``` cs
PublicDependencyModuleNames.AddRange(
    new string[] {
        "Core",
        "CoreUObject",
        "Engine",
        "InputCore",
        "UMG"
    }
);
```

并取消以下行的注释：

``` cs    
// Uncomment if you are using Slate UI
PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
```

修改后的完整文件如下：

``` cs    
// Fill out your copyright notice in the Description page of Project Settings.

using UnrealBuildTool;

public class YourProject : ModuleRules
{
    public YourProject(TargetInfo Target)
    {
        PublicDependencyModuleNames.AddRange(new string[] { 
            "Core", 
            "CoreUObject", 
            "Engine", 
            "InputCore", 
            "UMG"
            }
        );

        PrivateDependencyModuleNames.AddRange(new string[] {  });

        // Uncomment if you are using Slate UI
        PrivateDependencyModuleNames.AddRange(new string[] { 
            "Slate", 
            "SlateCore" 
            }
        );

        // Uncomment if you are using online features
        // PrivateDependencyModuleNames.Add("OnlineSubsystem");

        // To include OnlineSubsystemSteam, add it to the plugins section in your uproject file with the Enabled attribute set to true
    }
}
```

----------

## **<font color=#191970 size=5>UMG控件蓝图</font> ** ##

**1 <font color=#191970 size=4>创建控件蓝图</font>**

在工程中任意创建一个控件蓝图：

![界面控件](http://img.blog.csdn.net/20161125102434227)

接着在控件中的CanvasPanel中创建一个ProgressBar

![ProgressBar](http://img.blog.csdn.net/20161125102939579)

任意调整下细节设置，我的细节设置如下：

![细节设置](http://img.blog.csdn.net/20161125105534097)

一个蓝图控件就创建完成了，我们将控件命名为<font color=#dd1144 size=3>HpBar</font>

**2 <font color=#191970 size=4>添加到WidgetComponent</font>**

往任意Actor上添加一个新的WidgetComponent：

![添加WidgetComponent](http://img.blog.csdn.net/20161125112349537)

设置WidgetComponent如下图所示：

![设置WidgetComponent](http://img.blog.csdn.net/20161125112931523)

这里也有几个参数要讲一下：

- Space 有两种方式<font color=#dd1144 size=3>World</font>和<font color=#dd1144 size=3>Screen</font>
 > <font color=#dd1144 size=3>World</font>方式是绘制在场景中，会被物体遮挡，相当于一个场景的公告板。<br>
 > <font color=#dd1144 size=3>Screen</font>方式是绘制在屏幕上，不会被物体遮挡，并且一直面向摄像机。<br>

- Widget Class 就是选择你要渲染的控件，选择刚才你创建的空间蓝图。
- Draw Size 就是你要绘制的尺寸，根据你的需要调整。

调整完后我们来测试一下，就会发现控件已经正常显示了：

![示例图片](http://img.blog.csdn.net/20161125114330279)

----------

## **<font color=#191970 size=5>UUserWidget的UMG控件</font> ** ##

>**<font color=#DC143C size=4>此方式需要使用C++</font>** 

在你的Actor类中添加该属性

- <font color=#dd1144 size=3>YourActor.h</font>

``` cpp
// WidgetComponent
UPROPERTY(EditAnywhere, Category = WidgetComponent)
class UWidgetComponent* WidgetComponent;
```

- <font color=#dd1144 size=3>YourActor.cpp</font>

``` cpp
YourActor::YourActor(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{
    
    // ....你的代码

    // 初始化WidgetComponent
    WidgetComponent = CreateDefaultSubobject<UWidgetComponent>(TEXT("WidgetComponent"));
    WidgetComponent->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform);

    // 设置控件蓝图
    UClass* Widget = LoadClass<UUserWidget>(NULL, TEXT("WidgetBlueprint'/Game/UI/widget/HpBar.HpBar_C'"));
    WidgetComponent->SetWidgetClass(Widget);

    // 设置User Widget
    WidgetComponent->SetWidgetClass(MyUserWidget::StaticClass());
}
```

这里值得注意的是<font color=#dd1144 size=3>SetWidgetClass</font>这个方法。

- 如果你需要加载的是<font color=#dd1144 size=3>控件蓝图</font>

    就要像实例中使用<font color=#dd1144 size=3>LoadClass</font>模板函数，加载Class后进行设置。

``` cpp
// 设置控件蓝图
UClass* Widget = LoadClass<UUserWidget>(NULL, TEXT("WidgetBlueprint'/Game/UI/widget/HpBar.HpBar_C'"));
WidgetComponent->SetWidgetClass(Widget);
```

- 如果你需要加载的是一个继承自<font color=#dd1144 size=3>UUserWidget</font>的C++类

    你可以像下面这样直接设置：

``` cpp
// 设置User Widget
WidgetComponent->SetWidgetClass(MyUserWidget::StaticClass());
```

----------

## **<font color=#191970 size=5>SWidget的Slate控件</font> ** ##

>**<font color=#DC143C size=4>此方式需要使用C++</font>** 

在你的Actor类中添加该属性，这里我用<font color=#dd1144 size=3>SProgressBar</font>举例，原理相同，换成你想要的继承自<font color=#dd1144 size=3>SWidget</font>的<font color=#dd1144 size=3>Slate</font>控件就可以了，重写<font color=#dd1144 size=3>Slate</font>：

- <font color=#dd1144 size=3>YourActor.h</font>

``` cpp
// 在初始化完组件之后调用
virtual void PostInitializeComponents() override;

// Slate Widget
TSharedPtr<class SProgressBar> CurrentSlateWidget;
```

- <font color=#dd1144 size=3>YourActor.cpp</font>

``` cpp
void AMyCharacter::PostInitializeComponents()
{
    CurrentSlateWidget = SNew(SProgressBar);
    
    if ( CurrentSlateWidget.IsValid() )
    {
        WidgetComponent->SetSlateWidget(CurrentSlateWidget);
        // WidgetComponent->SetWidgetSpace(EWidgetSpace::Screen);
    }
}
```

>**<font color=#DC143C size=4>这里有个大坑，可能是WidgetComponent设计漏洞，只要用Slate Widget，就不能使用屏幕绘制模式（EWidgetSpace::Screen），使用就会崩溃。</font>**

如果用<font color=#dd1144 size=3>Slate</font>制作的血条不能朝向摄像机，那真是太蛋疼了。<br>
最后，我选择了旋转<font color=#dd1144 size=3>WidgetComponent</font>的方式来解决这个问题。

----------

## **<font color=#191970 size=5>扩展WidgetComponent</font> ** ##

创建一个新类继承自WidgetComponent的新组建，取名为<font color=#dd1144 size=3>MyWidgetComponent</font>。

- <font color=#dd1144 size=3>MyWidgetComponent.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "Components/WidgetComponent.h"
#include "MyWidgetComponent.generated.h"

/**
 * 
 */
UCLASS(Blueprintable, ClassGroup="UserInterface", hidecategories=(Object,Activation,"Components|Activation",Sockets,Base,Lighting,LOD,Mesh), editinlinenew, meta=(BlueprintSpawnableComponent) )
class TESTMOBILE_API UMyWidgetComponent : public UWidgetComponent
{
    GENERATED_BODY()
    
public:
    // 设置该角色属性的默认值
    UMyWidgetComponent(const FObjectInitializer& ObjectInitializer = FObjectInitializer::Get());

    virtual void TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction *ThisTickFunction) override;

protected:

    /** Should we Toward Camera */
    UPROPERTY(EditAnywhere, Category=UserInterface)
    bool bTowardCamera;
};
```

在类体重增加了一个是否朝向摄像机的属性，然后重写父类的<font color=#dd1144 size=3>TickComponent</font>方法，实现如下：

- <font color=#dd1144 size=3>MyWidgetComponent.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#include "Kismet/GameplayStatics.h"
#include "MyWidgetComponent.h"


// 设置默认属性
UMyWidgetComponent::UMyWidgetComponent(const FObjectInitializer& ObjectInitializer /*= FObjectInitializer::Get()*/)
    : Super(ObjectInitializer)
    , bTowardCamera(true)
{
}


// Tick函数
void UMyWidgetComponent::TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction *ThisTickFunction)
{
    Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

    if (Space != EWidgetSpace::Screen  && bTowardCamera)
    {

        FRotator WidgetComponentRotator = GetComponentRotation();
        FRotator CameraRotator = UGameplayStatics::GetPlayerCameraManager(this, 0)->GetCameraRotation();

        this->SetWorldRotation(FRotator(-CameraRotator.Pitch, CameraRotator.Yaw+180, WidgetComponentRotator.Roll));

    }
}
```

在<font color=#dd1144 size=3>TickComponent</font>方法中，获取了当前摄像机的旋转信息，并根据旋转信息，并给<font color=#dd1144 size=3>MyWidgetComponent</font>了一个朝向摄像机的旋转。<br>

![示例图片](http://img.blog.csdn.net/20161125114330279)

现在，你的<font color=#dd1144 size=3>Slate</font>控件可以朝向摄像机了。

备注：
关于方法传入的Pitch-Yaw—Roll可以参考[Unreal Engine 4 C++ Camera Pitch Yaw Roll 直观理解](http://blog.csdn.net/qq_20309931/article/details/53375517)<br>
关于头顶血条的详细方法可以参考[Unreal Engine 4 C++ UMG血条及头顶信息](http://blog.csdn.net/qq_20309931/article/details/53338714)

搞定，这样就可以让Slate控件朝向摄像机。
