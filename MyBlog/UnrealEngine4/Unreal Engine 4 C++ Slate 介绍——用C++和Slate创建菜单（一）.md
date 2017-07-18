[TOC]

# **Unreal Engine 4 C++ Slate 介绍——用C++和Slate创建菜单（一）**#

好记性不如烂笔头啊，还是记录一下!

----------

这是教程的第一部分，会学习一些基础的东西。我们将创建一个非常基本的、没有任何样式的游戏菜单，只会简单的显示一个游戏标题，并提供两个按钮：<font color=#dd1144 size=3>Play Game</font>和<font color=#dd1144 size=3>Quit</font>。在下一个教程中，我将开始实现可以在虚幻编辑器中编辑的<font color=#dd1144 size=3>Slate UI Styles</font>，允许您在编辑器本身内调整菜单的外观和感觉。接下来，我们将开始进入数据绑定，这对于更新UI的数据非常有用 - 例如，如果您要在商店界面中设置页面，则可能需要一个文本块显示玩家目前在她的购物车有多少项物品。最后，我将以可用于创建更多动态，交互式菜单的方法结束，例如您在<font color=#dd1144 size=3>StrategyGame</font>示例中看到的。

## **<font color=#191970 size=5>Slate的准备工作</font>** ##

这个项目中的示例中，我将假设您已经创建一个空白项目。有没有初学者内容不重要，我们并不会在本教程中使用它。如果你创建了你的项目，先为项目生成代码和Visual Studio或Xcode项目。本教程系列的剩余部分，我假定您的项目名称为<font color=#dd1144 size=3>SlateTutorials</font>的情况下运行 - 请记住在适当的情况下将其替换为您自己的项目名称。在项目的源文件夹中，打开SlateTutorials.Build.cs文件，取消注释或添加以下行：

``` cs
PrivateDependencyModuleNames.AddRange(
    new string[] {
        "Slate",
        "SlateCore"
    }
);
```

当您尝试构建项目时，将添加必要的Slate库和头文件到您的项目路径。您现在可以编写您的第一个Slate UI！

----------

## **<font color=#191970 size=5>使用AHUD</font> ** ##

我们要做的第一件事情是向项目中添加一个新的HUD类，我们取名为MainMenuHUD：

- MainMenuHUD.h 

``` cpp

#include "GameFramework/HUD.h"
#include "MainMenuHUD.generated.h"
 
/**
  * Provides an implementation of the game’s Main Menu HUD, which will embed and respond to events triggered
  * within SMainMenuUI.
  */
UCLASS()
class SLATETUTORIALS_API AMainMenuHUD : public AHUD
{
    GENERATED_BODY()
    // Initializes the Slate UI and adds it as widget content to the game viewport.
    virtual void PostInitializeComponents() override;
 
    // Reference to the Main Menu Slate UI.
    TSharedPtr<class SMainMenuUI> MainMenuUI;
 
public:
    // Called by SMainMenu whenever the Play Game! button has been clicked.
    UFUNCTION(BlueprintImplementableEvent, Category = "Menus|Main Menu")
    void PlayGameClicked();
 
    // Called by SMainMenu whenever the Quit Game button has been clicked.
    UFUNCTION(BlueprintImplementableEvent, Category = "Menus|Main Menu")
    void QuitGameClicked();
};
```

- MainMenuHUD.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
#include "MainMenuHUD.h"
 
void AMainMenuHUD::PostInitializeComponents()
{
    Super::PostInitializeComponents();
}
```

我觉得这里的注释对一切做了很好的解释，并且如果你经常使用C++和Slate，对这部分应该已经非常熟悉了。如果你还没想好<font color=#dd1144 size=3>PlayGameClicked()</font>和<font color=#dd1144 size=3>QuitGameClicked()</font>要做什么，你需要给他们加上<font color=#dd1144 size=3>UFUNCTION</font>宏，这两个函数将是任意蓝图继承<font color=#dd1144 size=3>MainMenuHUD</font>类就会触发的事件，我们可以用于做任何事情。比如说，我可以在使用<font color=#dd1144 size=3>SMainMenuUI</font>时调用这两个函数。我看了很多教程（包括UE4 维基上的<font color=#dd1144 size=3>Slate, Hello!</font>教程，这个教程对我帮助很多）都是在<font color=#dd1144 size=3>BeginPlay()</font>函数来初始化UI。但我发现，在<font color=#dd1144 size=3>StrategyGame</font>示例中是在<font color=#dd1144 size=3>PostInitializeComponents()</font>函数中进行的初始化，我选择使用后者的方法来实现。

----------

## ** <font color=#191970 size=5>创建主菜单窗口</font> ** ##

现在，我们需要往工程中添加另一个类：<font color=#dd1144 size=3>SMainMenuUI</font>类，继承与<font color=#dd1144 size=3>SCompoundWidget</font>类，这个类要比HUD类复杂，所以我先展示代码，后面再详细解释每个部分。

- MainMenuUI.h 

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
/**
  * MainMenuUI.h – Provides an implementation of the Slate UI representing the main menu.
  */
 
#pragma once
 
#include "SlateBasics.h"
#include "MainMenuHUD.h"
 
// Lays out and controls the Main Menu UI for our tutorial.
class SLATETUTORIALS_API SMainMenuUI : public SCompoundWidget
{
 
public:
    SLATE_BEGIN_ARGS(SMainMenuUI)
    {}
    SLATE_ARGUMENT(TWeakObjectPtr<class AMainMenuHUD>, MainMenuHUD)
    SLATE_END_ARGS()
 
    // Constructs and lays out the Main Menu UI Widget.
    // args Arguments structure that contains widget-specific setup information.
    void Construct(const FArguments& args);
 
    // Click handler for the Play Game! button – Calls MenuHUD’s PlayGameClicked() event.
    FReply PlayGameClicked();
 
    // Click handler for the Quit Game button – Calls MenuHUD’s QuitGameClicked() event.
    FReply QuitGameClicked();
 
    // Stores a weak reference to the HUD controlling this class.
    TWeakObjectPtr<class AMainMenuHUD> MainMenuHUD;
};
```

大部分代码是很直接的。注意：并没有将类指定为<font color=#dd1144 size=3>UCLASS()</font>，实际上这个类并不需要到处给蓝图。首先这个类继承自<font color=#dd1144 size=3>SCompoundWidget</font>，<font color=#dd1144 size=3>SCompoundWidget</font>是一个Slate控件，可以由其他控件组成——在Slate的API中有很多示例：SVerticalBox，SHorizontalBox，SOverlay等等。相反的控件是<font color=#dd1144 size=3>SLeafWidget</font>，它不包含任何控件。

``` cpp
    
    SLATE_BEGIN_ARGS(SMainMenuUI)
    {}
    
    SLATE_ARGUMENT(TWeakObjectPtr<class AMainMenuHUD>, MainMenuHUD)

    SLATE_END_ARGS()

    // Constructs and lays out the Main Menu UI Widget.
    // args Arguments structure that contains widget-specific setup information.
    void Construct(const FArguments& args);

```

如果你没有习惯运用虚幻的宏，这部分可能看起来很奇怪。这三个宏的作用是用来生成一个结构，其中包含在构建过程中的参数列表。您可能注意到了<font color=#dd1144 size=3>Construct</font>函数的参数<font color=#dd1144 size=3>FArguments</font>——这些宏就是在定义这个结构。在我们的示例中，我们只有一个参数，一个指向一个拥有这个类的<font color=#dd1144 size=3>AMainMenuHUD</font>弱指针。如果你还不熟悉智能指针，这简单的意味着这个指针可以引用父类HUD，但是避免了一个强引用的循环（记住，我们的HUD类也引用了这个对象），强引用循环会导致即使这些引用已经不在使用的时候对象依然存在在内存中无法释放。

``` cpp
void Construct(const FArguments& args);
```

<font color=#dd1144 size=3>Construct()</font>方法接收一个参数，我们使用<font color=#dd1144 size=3>SLATE_*</font>宏对<font color=#dd1144 size=3>FArguments</font>结构化，结构中包含了控件的所有参数。当我们创建控件时，将调用这个方法，您很快就可以看到如何布置窗口控件。

``` cpp
// Click handler for the Play Game! button – Calls MenuHUD’s PlayGameClicked() event.
FReply PlayGameClicked();
 
// Click handler for the Quit Game button – Calls MenuHUD’s QuitGameClicked() event.
FReply QuitGameClicked();
```

这个两个方法是用来处理我们将要添加的按钮<font color=#dd1144 size=3>Play Game</font>和<font color=#dd1144 size=3>Quit Game</font>，这两个函数整合<font color=#dd1144 size=3>FOnClicked</font>事件，它只返回一个<font color=#dd1144 size=3>FReply</font>（告诉引擎是否处理事件）不接收任何参数。如果我们想要的话，我们可以为这些方法指定一个参数。在后面的教程，我会教你如何绑定数据并且使用他们。<br>

<font color=#191970 size=4>继续我们的主菜单</font><br>

现在，我们需要去实现我们的主菜单控件。这将是一个相当容易的任务，因为我们只需要实现三个方法——但他们在刚开始来看是非常的奇怪（当用Slate进行工作时，我推荐你用良好的习惯保持缩进，缩进对于理解布局是非常有必要的）。

- MainMenuUI.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
#include "MainMenuUI.h"
#include "Engine.h"
 
BEGIN_SLATE_FUNCTION_BUILD_OPTIMIZATION
void SMainMenuUI::Construct(const FArguments& args)
{
    MainMenuHUD = args._MainMenuHUD;
 
    ChildSlot
    [
        SNew(SOverlay)
        + SOverlay::Slot()
        .HAlign(HAlign_Center)
        .VAlign(VAlign_Top)
        [
            SNew(STextBlock)
            .ColorAndOpacity(FLinearColor::White)
            .ShadowColorAndOpacity(FLinearColor::Black)
            .ShadowOffset(FIntPoint(-1, 1))
            .Font(FSlateFontInfo("Arial", 26))
            .Text(FText::FromString("Main Menu"))
        ]
        + SOverlay::Slot()
        .HAlign(HAlign_Right)
        .VAlign(VAlign_Bottom)
        [
            SNew(SVerticalBox)
            + SVerticalBox::Slot()
            [
                SNew(SButton)
                .Text(FText::FromString("Play Game!"))
                .OnClicked(this, &SMainMenuUI::PlayGameClicked)
            ]
            + SVerticalBox::Slot()
            [
                SNew(SButton)
                .Text(FText::FromString("Quit Game"))
                .OnClicked(this, &SMainMenuUI::QuitGameClicked)
            ]
        ]
    ];
 
}
END_SLATE_FUNCTION_BUILD_OPTIMIZATION
 
FReply SMainMenuUI::PlayGameClicked()
{
    if (GEngine)
    {
        GEngine->AddOnScreenDebugMessage(-1, 3.f, FColor::Yellow, TEXT("PlayGameClicked"));
    }
 
    // actually the BlueprintImplementable function of the HUD is not called; uncomment if you want to handle the OnClick via Blueprint
    //MainMenuHUD->PlayGameClicked();
    return FReply::Handled();
}
 
FReply SMainMenuUI::QuitGameClicked()
{
    if (GEngine)
    {
        GEngine->AddOnScreenDebugMessage(-1, 3.f, FColor::Yellow, TEXT("QuitGameClicked"));
    }
 
    // actually the BlueprintImplementable function of the HUD is not called; uncomment if you want to handle the OnClick via Blueprint
    //MainMenuHUD->QuitGameClicked();
    return FReply::Handled();
}
```

你可以看到我所说的定义Slate布局，关于缩进对于尴尬的布局的作用。我们从两个事件处理函数看起。如前面所讲，他们首先会调用HUD上的蓝图事件，这样便可以使我们处理蓝图对这些按钮的点击的响应，然后它们返回了<font color=#dd1144 size=3>FReply::Handled()</font>——这是让引擎知道我们已经处理了点击事件，所以它不需要对玩家的鼠标输入做任何其他处理。<br>

我们再来看看<font color=#dd1144 size=3>Construct()</font>方法，如前面所讲，<font color=#dd1144 size=3>FArguments</font>是用<font color=#dd1144 size=3>SLATE*</font>宏定义的。在我们的示例中，它添加了一个方法，我们稍后将使用它来指定拥有此控件的<font color=#dd1144 size=3>HUD</font>。我们首先要做的事情是捕获这个值，我们只需要将本地值设置为args包含的值就可以了。注意：args里面的实际名称是<font color=#dd1144 size=3>_MainMenuHUD</font>，而不是<font color=#dd1144 size=3>MainMenuHUD</font>，<font color=#dd1144 size=3>_MainMenuHUD</font>是实际设置的变量，所有通过<font color=#dd1144 size=3>SLATE_ARGUMENT</font>宏传递的参数都遵循此规定。<br>

整个布局定义是由Epic写的一个非常漂亮的地方，就是利用运算符重载和其他酷炫功能来定义我们的UI布局。简单的说，我们使用<font color=#dd1144 size=3>[]</font>运算符来定义作为我们定制（组合）控件的子节点控件。对于组合控件，我需要先调用<font color=#dd1144 size=3>SNew(WidgetClass)</font>方法，然后使用<font color=#dd1144 size=3>+ WidgetClass::Slot()</font>来增加控件。然后我们可以在该<font color=#dd1144 size=3>slot</font>中使用<font color=#dd1144 size=3>[]</font>为该插槽指定子项。<br>

<font color=#dd1144 size=3>SOverlay</font>的第一个子控件是<font color=#dd1144 size=3>STextBlock</font>，他与自他GUI API中的<font color=#dd1144 size=3>TextBlock</font>或者<font color=#dd1144 size=3>Label</font>非常相似。为此，我们指定了颜色、阴影颜色、阴影偏移量、字体和一些文本。所有这些都是相当显而易见的，但是请注意一下，我们在<font color=#dd1144 size=3>SNew</font>后没有任何<font color=#dd1144 size=3>::Slot()</font>调用，因为<font color=#dd1144 size=3>TextBlock</font>是一个<font color=#dd1144 size=3>SLeafWidget</font>，它不能包含子控件。所以我们没有任何插槽可以使用，因此我们只能指定自身属性。<br>

我们的第二个子控件是一个组合控件——<font color=#dd1144 size=3>SVerticalBox</font>。垂直框（与它相对应的是水平框），将所有元素由上到下依次排列（对于水平框是从左到右），占用相同的控件。在<font color=#dd1144 size=3>SVerticalBox</font>的插槽内（记住，<font color=#dd1144 size=3>SVerticalBox</font>是个组合控件），我们指定两个参数的<font color=#dd1144 size=3>SButton</font>实例。第一个参数是显示在按钮上的文本，第二个是事件处理的绑定函数（PlayGameClicked / QuitGameClicked），每当点击时会调用该函数。然后我们完成了布局的编码，记得用;号结尾<br>

----------

## ** <font color=#191970 size=5>重新修改HUD</font> ** ##

我们已经完成了主菜单的布局设置。是时候就把它绑定到我们的HUD上！返回调整<font color=#dd1144 size=3>PostInitializeComponents</font>方法：

- MainMenuHUD.cpp 

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
#include "MainMenuHUD.h"
#include "MainMenuUI.h"
#include "Engine.h"
 
void AMainMenuHUD::PostInitializeComponents()
{
    Super::PostInitializeComponents();
 
    SAssignNew(MainMenuUI, SMainMenuUI).MainMenuHUD(this);
 
    if (GEngine->IsValidLowLevel())
    {
        GEngine->GameViewport->AddViewportWidgetContent(SNew(SWeakWidget).PossiblyNullContent(MainMenuUI.ToSharedRef()));
    }
}
```

这里的设置就比较简单了——在确保<font color=#dd1144 size=3>Engine</font>和<font color=#dd1144 size=3>Viewport</font>有效后，我们创建一个<font color=#dd1144 size=3>MainMenuUI</font>的实例，然后将控件作为内容添加到游戏视图上！注意：<font color=#dd1144 size=3>SAssignNew</font>添加<font color=#dd1144 size=3>MenuHUD</font>——这又是我们<font color=#dd1144 size=3>SLATE_</font>宏的结果，记得前面我们提到的<font color=#dd1144 size=3>SLATE_ARGUMENT</font>宏的<font color=#dd1144 size=3>MenuHUD</font>部分？它不仅设置了我们的变量（<font color=#dd1144 size=3>_MainMenuHUD</font>），也同时生成了我们这里用到的<font color=#dd1144 size=3>MainMenuHUD(this)</font>这个设置方法。<br>

----------

## ** <font color=#191970 size=5>设置游戏模式</font> ** ##

使用虚幻创建一个游戏模式继承自<font color=#dd1144 size=3>GameMode</font>取名为WidgetGameMode，并修改成一下样子：

- WidgetGameMode.h 

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#pragma once
 
#include "GameFramework/GameMode.h"
#include "WidgetGameMode.generated.h"
 
UCLASS()
class SLATETUTORIALS_API AWidgetGameMode : public AGameMode
{
    GENERATED_UCLASS_BODY()
 
public:
    AWidgetGameMode();
 
};
```

- WidgetGameMode.cpp 

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
#include "WidgetGameMode.h"
#include "MainMenuHUD.h"
 
AWidgetGameMode::AWidgetGameMode()
{
    HUDClass = AMainMenuHUD::StaticClass();
}
```

然后在项目设置里选择<font color=#dd1144 size=3>Maps & Modes</font>

![GameMode Seting](https://d26ilriwvtzlb.cloudfront.net/8/85/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82_2015-03-18_13.21.16.png)

----------

## ** <font color=#191970 size=5>总结</font> ** ##

在你的游戏模式上设置适当的游戏模式类，设置你的关卡使用你的新游戏模式，并运行！ 如果一切顺利，你应该会显示如下图，运行正常（但有点丑陋）的游戏菜单！ 不要担心，在下一个教程中，我们将开始设置样式，这将允许我们大幅改善菜单项的外观！

![运行示例](https://d26ilriwvtzlb.cloudfront.net/1/16/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82_2015-03-18_13.28.24.png)
