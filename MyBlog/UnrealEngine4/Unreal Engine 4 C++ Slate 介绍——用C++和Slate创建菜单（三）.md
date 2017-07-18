[TOC]

# **Unreal Engine 4 C++ Slate 介绍——用C++和Slate创建菜单（三）**#

好记性不如烂笔头啊，还是记录一下!

----------

欢迎回到我的使用Slate和C++在虚幻引擎4中创建菜单的教程系列！

----------

## **<font color=#191970 size=5>第一步：概述和准备</font>** ##

什么是数据绑定？ 数据绑定是一种来自软件开发的概念，其中信息输出（例如玩家的当前血量）与实际展示的信息相关联。这样，只要更改数据（例如，对玩家造成伤害），显示就会自动更新。<br>

在我们的示例中，我们将创建一个新的<font color=#dd1144 size=3>Slate UI</font>，用作游戏中的HUD，在屏幕的上角显示玩家的当前血量和得分。最初，HUD将只有静态值 - 我们将在下一步中将其更改为绑定数据。 我将粘贴下面的代码，但不会详细介绍它是如何工作的 - 这是所有很简单的内容，已经在过去的教程中已经讲过了。

- TutorialGameHUDUI.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// TutorialGameHUDUI.h - Provides an implementation of the Slate UI representing the tutorial game HUD.
 
#pragma once
 
#include "Slate.h"
 
// Lays out and controls the Tutorial HUD UI.
 
class STutorialGameHUDUI : public SCompoundWidget
{
    SLATE_BEGIN_ARGS(STutorialGameHUDUI)
        : _OwnerHUD()
    {
    }
 
    SLATE_ARGUMENT(TWeakObjectPtr<class ATutorialGameHUD>, OwnerHUD);
 
    SLATE_END_ARGS()
 
public:
    /**
     * Constructs and lays out the Tutorial HUD UI Widget.
     * 
     * \args Arguments structure that contains widget-specific setup information.
     **/
    void Construct(const FArguments& args);
 
private:
    /**
     * Stores a weak reference to the HUD owning this widget.
     **/
    TWeakObjectPtr<class ATutorialGameHUD> OwnerHUD;
 
    /**
     * A reference to the Slate Style used for this HUD's widgets.
     **/
    const struct FGlobalStyle* HUDStyle;
};
```

- TutorialGameHUDUI.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
 
#include "TutorialGameHUD.h"
#include "TutorialGameHUDUI.h"
 
#include "Menus/GlobalMenuStyle.h"
#include "Menus/MenuStyles.h"
 
void STutorialGameHUDUI::Construct(const FArguments& args)
{
    OwnerHUD = args._OwnerHUD;
 
    HUDStyle = &FMenuStyles::Get().GetWidgetStyle<FGlobalStyle>("Global");
 
    ChildSlot
        [
            SNew(SOverlay)
            + SOverlay::Slot()
                .HAlign(HAlign_Right)
                .VAlign(VAlign_Top)
                [
                    SNew(STextBlock)
                        .TextStyle(&HUDStyle->MenuTitleStyle)
                        .Text(FText::FromString("SCORE: 0"))
                ]
            + SOverlay::Slot()
                .HAlign(HAlign_Left)
                .VAlign(VAlign_Top)
                [
                    SNew(STextBlock)
                        .TextStyle(&HUDStyle->MenuTitleStyle)
                        .Text(FText::FromString("HEALTH: 100"))
                ]
        ];
}
```

- TutorialGameHUD.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// TutorialGameHUD.h - Provides an implementation of the HUD that will embed the Tutorial Game UI.
 
#pragma once
 
#include "GameFramework/HUD.h"
 
#include "TutorialGameHUD.generated.h"
 
/**
 * Provides an implementation of the game's in-game HUD, which will display the player's current health and score.
 **/
UCLASS()
class ATutorialGameHUD : public AHUD
{
    GENERATED_UCLASS_BODY()
 
public:
    
    /**
     * Initializes the Slate UI and adds it as a widget to the game viewport.
     **/
    virtual void PostInitializeComponents() override;
 
private:
    
    /**
     * Reference to the Game HUD UI.
     **/
    TSharedPtr<class STutorialGameHUDUI> GameHUD;
};
```

- TutorialGameHUD.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
 
#include "TutorialGameHUD.h"
#include "TutorialGameHUDUI.h"
 
ATutorialGameHUD::ATutorialGameHUD(const class FPostConstructInitializeProperties& PCIP)
    : Super(PCIP)
{
}
 
void ATutorialGameHUD::PostInitializeComponents()
{
    Super::PostInitializeComponents();
 
    if (GEngine && GEngine->GameViewport)
    {
        UGameViewportClient* Viewport = GEngine->GameViewport;
 
        SAssignNew(GameHUD, STutorialGameHUDUI)
            .OwnerHUD(TWeakObjectPtr<ATutorialGameHUD>(this));
 
        Viewport->AddViewportWidgetContent(
            SNew(SWeakWidget).PossiblyNullContent(GameHUD.ToSharedRef())
            );
    }
}
```

继续构建项目并设置一个新的地图和游戏模式，并让HUD显示出来！

----------

## **<font color=#191970 size=5>第二步：绑定数据</font>** ##

我们有两个信息需要显示，我们想绑定到我们的UI：得分和血量。这两个都是整数，但它们必须绑定为我们的HUD的字符串！我们很快会从游戏模式（积分）和角色（血量）获取此信息，但首先我们将处理数据绑定部分。我们的绑定将有两个重要的任务：第一，它将获取实际数据。接下来，它会将其转换为要应用于文本块控件的FText。<br>

他们都需要两个东西：属性和绑定它的东西。在我们的例子中，因为我们需要对数据进行额外的处理（从一个整数转换为一个字符串），我们将在我们的widget类本身有一个函数来绑定。 将以下私有属性值和方法添加到<font color=#dd1144 size=3>STutorialGameHUDUI</font>类中：

- TutorialGameHUDUI.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// TutorialGameHUDUI.h - Provides an implementation of the Slate UI representing the tutorial game HUD.
 
#pragma once
 
#include "Slate.h"
 
// Lays out and controls the Tutorial HUD UI.
 
class STutorialGameHUDUI : public SCompoundWidget
{
    SLATE_BEGIN_ARGS(STutorialGameHUDUI)
        : _OwnerHUD()
    {
    }
 
    SLATE_ARGUMENT(TWeakObjectPtr<class ATutorialGameHUD>, OwnerHUD);
 
    SLATE_END_ARGS()
 
public:
    /**
     * Constructs and lays out the Tutorial HUD UI Widget.
     * 
     * \args Arguments structure that contains widget-specific setup information.
     **/
    void Construct(const FArguments& args);
 
private:
    /**
     * Stores a weak reference to the HUD owning this widget.
     **/
    TWeakObjectPtr<class ATutorialGameHUD> OwnerHUD;
 
    /**
     * A reference to the Slate Style used for this HUD's widgets.
     **/
    const struct FGlobalStyle* HUDStyle;

private:
    /**
     * Attribute storing the binding for the player's score.
     **/
    TAttribute<FText> Score;
 
    /**
     * Attribute storing the binding for the player's health.
     **/
    TAttribute<FText> Health;
 
    /**
     * Our Score will be bound to this function, which will retrieve the appropriate data and convert it into an FText.
     **/
    FText GetScore() const;
 
    /**
     * Our Health will be bound to this function, which will retrieve the appropriate data and convert it into an FText.
     **/
    FText GetHealth() const;    
};
```

在虚幻中使用<font color=#dd1144 size=3>TAttribute</font>类型来提供有一个<font color=#dd1144 size=3>accessor/getter</font>的数据绑定。 接下来，我们有两个常量函数，负责检索和格式化数据到UI可以使用的类型！那么我们如何实际做绑定？ 好吧，它很简单 - 事实上，如果你在C++中为Unreal项目完成了输入绑定，你已经做到了。 在STutorialGameHUDUI的Construct方法的顶部，在捕获HUDStyle之后，添加以下内容将我们的<font color=#dd1144 size=3>TAttributes</font>绑定到它们适当的函数：

``` cpp
Score.Bind(this, &STutorialGameHUDUI::GetScore);
Health.Bind(this, &STutorialGameHUDUI::GetHealth);
```

接下来，我们在UI布局中直接使用TAttributes类型：

``` cpp
+ SOverlay::Slot()
    .HAlign(HAlign_Right)
    .VAlign(VAlign_Top)
    [
        SNew(STextBlock)
            .TextStyle(&HUDStyle->MenuTitleStyle)
            .Text(Score)
    ]
+ SOverlay::Slot()
    .HAlign(HAlign_Left)
    .VAlign(VAlign_Top)
    [
        SNew(STextBlock)
            .TextStyle(&HUDStyle->MenuTitleStyle)
            .Text(Health)
    ]
```

最后，我们要在绑定函数放入一些临时数据，只是为了确保一切正常工作：

``` cpp
FText STutorialGameHUDUI::GetScore() const { return FText::FromString("SCORE: --"); }
FText STutorialGameHUDUI::GetHealth() const { return FText::FromString("HEALTH: --"); }
```

继续编译肯定一切正常，恭喜你！你刚刚绑定了你的文本块！ 您可以对Slate的所有内容执行此绑定 - 按钮文本，列表项，图像背景，样式等。

![示例图片](https://d26ilriwvtzlb.cloudfront.net/4/47/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82_2015-03-18_16.55.25.png)

----------

## **<font color=#191970 size=5>第三步：有用的数据</font>** ##

如果你只需要学习如何做数据绑定，而不关心本教程的细节，那么你可以跳过本节 - 在这里，我们只是实现了分数和血量功能。<br>

为了获得本教程的得分和血量数据，我添加了以下<font color=#dd1144 size=3>GameMode</font>和<font color=#dd1144 size=3>Character</font>类，并将它们设置为与<font color=#dd1144 size=3>GameMap</font>关卡一起使用。 

- TutGameMode.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// TutGameMode.h - Provides a simple game mode providing a Score!
 
#pragma once
 
#include "GameFramework/GameMode.h"
#include "TutGameMode.generated.h"
 
/**
 * A simple game mode providing a means of retrieving and adjusting a single Score value.
 **/
UCLASS()
class ATutGameMode : public AGameMode
{
    GENERATED_UCLASS_BODY()
 
public:
    
    /**
     * Retrieves the current Score from the game mode.
     **/
    UFUNCTION(BlueprintPure, BlueprintCallable, Category = "Score")
    int32 GetScore();
 
    /**
     * Adds to the game score.
     **/
    UFUNCTION(BlueprintCallable, Category = "Score")
    void AddPoints(int32 value);
 
    /**
     * Removes from the game score.
     **/
    UFUNCTION(BlueprintCallable, Category = "Score")
    void DeductPoints(int32 value);
 
private:
    
    /**
     * Stores the current score.
     **/
    int32 CurrentScore;
};
```

- TutGameMode.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
 
#include <algorithm>
 
#include "TutGameMode.h"
 
ATutGameMode::ATutGameMode(const class FPostConstructInitializeProperties& PCIP)
    : Super(PCIP), CurrentScore(0)
{
}
 
int32 ATutGameMode::GetScore()
{
    return CurrentScore;
}
 
void ATutGameMode::AddPoints(int32 value)
{
    if (value > 0)
        CurrentScore += value;
}
 
void ATutGameMode::DeductPoints(int32 value)
{
    if (value > 0)
        CurrentScore = std::max(CurrentScore - value, 0);
}
```

- TutorialCharacter.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// TutorialCharacter.h - Provides a simple character providing Health!
 
#pragma once
 
#include "GameFramework/Character.h"
#include "TutorialCharacter.generated.h"
 
/**
 * A simple character providing a means of retrieving and manipulating health.
 **/
UCLASS()
class ATutorialCharacter : public ACharacter
{
    GENERATED_UCLASS_BODY()
 
public:
    
    /**
     * Stores the character's current health.
     **/
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Combat")
    int32 Health;
};
```

- TutorialCharacter.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
 
#include "TutorialCharacter.h"
 
ATutorialCharacter::ATutorialCharacter(const class FPostConstructInitializeProperties& PCIP)
    : Super(PCIP)
{
    Health = 100;
}
```

一旦我们有了这些，更新得分和血量的数据是很简单的；

- TutorialGameHUDUI.cpp

``` cpp
FText STutorialGameHUDUI::GetScore() const
{
    // NOTE: THIS IS A TERRIBLE WAY TO DO THIS. DO NOT DO IT. IT ONLY WORKS ON SERVERS. USE GAME STATES INSTEAD!
    ATutGameMode* gameMode = Cast<ATutGameMode>(OwnerHUD->GetWorldSettings()->GetWorld()->GetAuthGameMode());
 
    if (gameMode == nullptr)
        return FText::FromString(TEXT("SCORE: --"));
 
    FString score = TEXT("SCORE: ");
    score.AppendInt(gameMode->GetScore());
 
    return FText::FromString(score);
}
 
FText STutorialGameHUDUI::GetHealth() const 
{
    ATutorialCharacter* character = Cast<ATutorialCharacter>(OwnerHUD->PlayerOwner->GetCharacter());
 
    if (character == nullptr)
        return FText::FromString(TEXT("HEALTH: --"));
 
    FString health = TEXT("HEALTH: ");
    health.AppendInt(character->Health);
 
    return FText::FromString(health);
}
```

继续并运行游戏，然后更新得分！（您可以在示例文件中通过按<font color=#dd1144 size=3>Home/End</font>来调整血量，然后按<font color=#dd1144 size=3>Page Up/Page Down</font>来调整分数）。示例文件下载地址：[File:SlateTutorials-3.zip](https://wiki.unrealengine.com/File:SlateTutorials-3.zip)
