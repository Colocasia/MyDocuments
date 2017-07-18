[TOC]

# **Unreal Engine 4 C++ Slate 介绍——用C++和Slate创建菜单（二）**#

好记性不如烂笔头啊，还是记录一下!

----------

欢迎来到教程的第二部分关于使用虚幻引擎4中的Slate和C++创建游戏菜单！在上一个教程中，我们使用Slate为我们的游戏创建了一个非常简单并且...普通的标题屏幕/主菜单。现在可能已经过时了，因为这种形式的菜单看起来更像我们的系统的应用程序菜单，而不是游戏应该有的华丽界面！今天，我们将通过引入样式来解决这个问题！使用样式后我们可以改变我们的通用“按钮”，变成像我们想要的样子！教程的资源可以在这下载：[UIpack RPG.zip](https://wiki.unrealengine.com/File:UIpack_RPG.zip)

----------

## **<font color=#191970 size=5>第一步：样式设置</font>** ##

首先要做的事是设置样式集和，它将用于加载和引用我们的样式。样式本身会在虚幻编辑器中指定，使我们能够对设计进行大量更改，而无需重新编译代码（我们目前必须为样式和布局更改都进行编译）。我们设置的Style Set，需要调用<font color=#dd1144 size=3>MenuStyles</font>，它是一个纯静态类。

- MenuStyles.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// MenuStyles.h - Provides our Style Set and allows us to load and reference UI Styles specified in-editor. 
 
#pragma once
 
#include "SlateBasics.h"
 
class FMenuStyles
{
public:
    // Initializes the value of MenuStyleInstance and registers it with the Slate Style Registry.
    static void Initialize();
 
    // Unregisters the Slate Style Set and then resets the MenuStyleInstance pointer.
    static void Shutdown();
 
    // Retrieves a reference to the Slate Style pointed to by MenuStyleInstance.
    static const class ISlateStyle& Get();
 
    // Retrieves the name of the Style Set.
    static FName GetStyleSetName();
 
private:
    // Creates the Style Set.
    static TSharedRef<class FSlateStyleSet> Create(); 
 
    // Singleton instance used for our Style Set.
    static TSharedPtr<class FSlateStyleSet> MenuStyleInstance;
};
```

这些方法注释已经阐明的很清楚了，看名字也大概知道作用。后面我们将在我们的<font color=#dd1144 size=3>Game Module</font>中调用<font color=#dd1144 size=3>Initialize()</font>和<font color=#dd1144 size=3>Shutdown()</font>方法。<font color=#dd1144 size=3>Get()</font>方法会在以后我们需要加载特定样式集时使用。<font color=#dd1144 size=3>GetStyleSetName()</font>用于引擎检索我们的样式集名称。这些方法的实现同样很简单：

- MenuStyles.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h"
#include "MenuStyles.h"
#include "SlateGameResources.h" 
 
TSharedPtr<FSlateStyleSet> FMenuStyles::MenuStyleInstance = NULL;
 
void FMenuStyles::Initialize()
{
    if (!MenuStyleInstance.IsValid())
    {
        MenuStyleInstance = Create();
        FSlateStyleRegistry::RegisterSlateStyle(*MenuStyleInstance);
    }
}
 
void FMenuStyles::Shutdown()
{
    FSlateStyleRegistry::UnRegisterSlateStyle(*MenuStyleInstance);
    ensure(MenuStyleInstance.IsUnique()); 
    MenuStyleInstance.Reset();
}
 
FName FMenuStyles::GetStyleSetName()
{
    static FName StyleSetName(TEXT("MenuStyles"));
    return StyleSetName;
}
 
TSharedRef<FSlateStyleSet> FMenuStyles::Create()
{
    TSharedRef<FSlateStyleSet> StyleRef = FSlateGameResources::New(FMenuStyles::GetStyleSetName(), "/Game/UI/Styles", "/Game/UI/Styles");
    return StyleRef;
}
 
const ISlateStyle& FMenuStyles::Get()
{
    return *MenuStyleInstance;
}
```

在<font color=#dd1144 size=3>Initialize()</font>中，我们判断MenuStyleInstance（我们的单例指针）是否有效（就是不为空）。如果不是有效的，我们就实例化它，并且用<font color=#dd1144 size=3>SlateStyleRegistry</font>来注册样式集。在<font color=#dd1144 size=3>Shutdown()</font>中，我们做出了相反的，我们取消了注册样式集，确保我们的指针是唯一的(在这种情况下应该是唯一的)，然后我们重置它（设置为空）
。<font color=#dd1144 size=3>GetStyleSetName()</font>中，我们只需将我们的样式的FName缓存为静态变量，并始终返回该值。 这样，我们就有一个简单的方式来获取我们的样式集单例。

----------

## **<font color=#191970 size=5>第二步：加入到Game Module</font>** ##

如果现在编译代码，它无法正常运行。我们还从来没有调用我们的静态方法！找到你的游戏模块的源文件（在我的工程方案中是SlateTutorials.cpp），里面应该真正只有两行：一个包括你的模块的头文件和类似于以下内容：
``` cpp
IMPLEMENT_PRIMARY_GAME_MODULE(FDefaultGameModuleImpl，SlateTutorials，“SlateTutorials”);
```
注意<font color=#dd1144 size=3>FDefaultGameModuleImpl</font>,这是用于你的游戏模块的类。很多时候不会在这里去处理任何其他事情 - 但我们需要绑定到游戏模块来初始化我们的样式集！我们如何做到这一点？好吧，Epic的做法（就是我们要使用的）似乎只是简单地定义模块类在这里 - 但要记住，如果你要做的比我们复杂很多，分为 <font color=#dd1144 size=3>header & source</font>是一个更好的主意。

``` cpp
#include "SlateTutorials.h"
#include "MenuStyles.h" 
 
//Custom implementation of the Default Game Module. 
class FSlateTutorialsGameModule : public FDefaultGameModuleImpl
{
    // Called whenever the module is starting up. In here, we unregister any style sets 
    // (which may have been added by other modules) sharing our 
    // style set's name, then initialize our style set. 
    virtual void StartupModule() override
    {
        //Hot reload hack
        FSlateStyleRegistry::UnRegisterSlateStyle(FMenuStyles::GetStyleSetName());
        FMenuStyles::Initialize();
    }
 
    // Called whenever the module is shutting down. Here, we simply tell our MenuStyles to shut down.
    virtual void ShutdownModule() override
    {
        FMenuStyles::Shutdown();
    }
 
};
 
IMPLEMENT_PRIMARY_GAME_MODULE(FSlateTutorialsGameModule, SlateTutorials, "SlateTutorials");
```

这里没什么难的，对吧？ 我们只是定义一个自定义的模块类，只是扩展我们以前的，并添加一些必要的调用来初始化和关闭我们的游戏模块。 注意，我们花时间去取消注册 - 只是为了防止任何其他模块引入一个相同的名称（老实说，我...不知道这是多么必要。但在<font color=#dd1144 size=3>Strategy</font>示例中是这么使用的，我想应该会有更好的办法）。

----------

## **<font color=#191970 size=5>第三步：创建一个Style类</font>** ##

现在我们有了样式集，让我们继续创建一个类，我们可以使用它来建立和自定义我们的菜单样式。你有很多很多方法做到这些，来满足您的布局需要。我个人倾向于有一个单一的“全局”样式定义的东西，例如标准按钮样式。然后创建控件特定的样式集，如果自定义控件（比如我们的主菜单UI）只存在一个或两个空格。 那么我们该怎么做呢？ 简单！ 我们将创建一个新的<font color=#dd1144 size=3>GlobalMenuStyle</font>类（和一个<font color=#dd1144 size=3>GlobalStyle</font>结构...你很快会看到）。

- GlobalMenuStyle.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
// GlobalMenuStyle.h - Provides a global menu style! 
 
#pragma once
 
#include "SlateWidgetStyleContainerBase.h"
#include "SlateWidgetStyle.h"
#include "SlateBasics.h"
#include "GlobalMenuStyle.generated.h" 
 
// Provides a group of global style settings for our game menus! 
USTRUCT()
struct FGlobalStyle : public FSlateWidgetStyle
{
    GENERATED_USTRUCT_BODY()
    // Stores a list of Brushes we are using (we aren't using any) into OutBrushes.
    virtual void GetResources(TArray<const FSlateBrush*>& OutBrushes) const override;
 
    // Stores the TypeName for our widget style.
    static const FName TypeName;
 
    // Retrieves the type name for our global style, which will be used by our Style Set to load the right file. 
    virtual const FName GetTypeName() const override;
 
    // Allows us to set default values for our various styles. 
    static const FGlobalStyle& GetDefault(); 
 
    // Style that define the appearance of all menu buttons. 
    UPROPERTY(EditAnywhere, Category = Appearance)
    FButtonStyle MenuButtonStyle;
 
    // Style that defines the text on all of our menu buttons. 
    UPROPERTY(EditAnywhere, Category = Appearance)
    FTextBlockStyle MenuButtonTextStyle;
 
    // Style that defines the text for our menu title. 
    UPROPERTY(EditAnywhere, Category = Appearance)
    FTextBlockStyle MenuTitleStyle;
};
 
// Provides a widget style container to allow us to edit properties in-editor
UCLASS(hidecategories = Object, MinimalAPI)
class UGlobalMenuStyle : public USlateWidgetStyleContainerBase
{
    GENERATED_UCLASS_BODY()
 
public:
    // This is our actual Style object. 
    UPROPERTY(EditAnywhere, Category = Appearance, meta = (ShowOnlyInnerProperties))
    FGlobalStyle MenuStyle;
 
    // Retrievs the style that this container manages. 
    virtual const struct FSlateWidgetStyle* const GetStyle() const override
    {
        return static_cast<const struct FSlateWidgetStyle*>(&MenuStyle);
    }
 
};
```

这一个有点长，但（像前面一样）不是很复杂。首先，我们有<font color=#dd1144 size=3>GetResources</font>方法 - 如果你使用任何Slate画刷（例如，定义一个SImage窗口控件的属性），你可以在这里用<font color=#dd1144 size=3>OutBrushes</font>添加这些画刷。 在我们的示例中，我们的按钮和文本块样式不是画刷，所以我们不必在这个方法中做任何事情。接下来，我们有<font color=#dd1144 size=3>GetTypeName</font>方法 - 这个方法给出了类型的名称，它应该匹配实际的类型名称。 这用于引用这个控件是什么类型。 <font color=#dd1144 size=3>GetDefault()</font>方法允许我们设置一些默认值 - 例如，如果我们想要的话我们可以为标题屏幕设置默认字体或大小。最后，我们有三个属性。第一和第二都是关于按钮 - 按钮（SButton控件）实际采取两种样式 - 一个用于按钮本身，一个用于表示按钮上的文本的文本块。第三个属性，然后是我们的菜单标题文本。但是，这一个结构是不够的！我们实际上有一个充当容器基础的类 - 这允许我们在编辑器中可以修改这些结构的公开属性，甚至比定义更简单的实现：

- GlobalMenuStyle.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#include "SlateTutorials.h" 
#include "GlobalMenuStyle.h" 
 
void FGlobalStyle::GetResources(TArray<const FSlateBrush*>& OutBrushes) const
{
}
 
const FName FGlobalStyle::TypeName = TEXT("FGlobalStyle");
 
const FName FGlobalStyle::GetTypeName() const
{
    static const FName TypeName = TEXT("FGlobalStyle");
    return TypeName;
}
 
const FGlobalStyle& FGlobalStyle::GetDefault()
{
    static FGlobalStyle Default;
    return Default;
}
 
UGlobalMenuStyle::UGlobalMenuStyle(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{
}
```

大多数这些方法是空的 - 毕竟，我们没有任何画刷注册，我同样不会设置默认值（字面上很简单，可以在<font color=#dd1144 size=3>GetDefault()</font>方法中更改您的样式的默认属性）。 只要确保你的<font color=#dd1144 size=3>GetTypeName()</font>返回一个FName匹配你的样式结构的名称！

![示例图片](https://d26ilriwvtzlb.cloudfront.net/5/53/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82_2015-03-17_17.20.36.png)

----------

## **<font color=#191970 size=5>第四步：定义你的风格</font>** ##

现在我们已经设置了样式，你不认为是时候去定义它们了吗？对吧！启动虚幻，在内容浏览器中创建一个名为UI的新文件夹，然后在其中创建一个名为Styles的新文件夹。 然后要创建实际的样式定义，创建一个新的<font color=#dd1144 size=3>Slate Widget</font>资源（<font color=#dd1144 size=3>用户界面</font>-><font color=#dd1144 size=3>Slate Widget Style</font>）。 将提示您选择控件样式容器 - 选择GlobalMenuStyle，并将新资产命名为Global（如果您名为其他名称，请记住您以后使用的名称）。继续打开它，并将您的属性调整到你喜欢的东西 - 随意导入一些图像，为您的按钮使用！

![示例图片](https://d26ilriwvtzlb.cloudfront.net/c/cc/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82_2015-03-17_17.18.21.png)

----------

## **<font color=#191970 size=5>第五步：使用你的风格</font>** ##

我们定义了我们的风格，设置了一些漂亮的设置，但是我们如何实际使用这个！<br>
首先，将以下成员变量添加到您的主菜单UI窗口控件：

``` cpp
const struct FGlobalStyle * MenuStyle;
```

然后，进入您的源文件，并添加两个标题包含：

``` cpp
#include "GlobalMenuStyle.h"
#include "MenuStyles.h"
```

- MainMenuUI.h

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
#pragma once
 
#include "SlateBasics.h"
 
class SLATETUTORIALS_API SMainMenuUI : public SCompoundWidget
{
 
public:
    SLATE_BEGIN_ARGS(SMainMenuUI)
    {}
    SLATE_ARGUMENT(TWeakObjectPtr<class AMainMenuHUD>, MainMenuHUD)
    SLATE_END_ARGS()
 
    void Construct(const FArguments& InArgs);
 
    /**
    * Click handler for the Play Game! button – Calls MenuHUD’s PlayGameClicked() event.
    */
    FReply PlayGameClicked();
    /**
    * Click handler for the Quit Game button – Calls MenuHUD’s QuitGameClicked() event.
    */
    FReply QuitGameClicked();
 
    TWeakObjectPtr<class AMainMenuHUD> MainMenuHUD;
 
    const struct FGlobalStyle* MenuStyle;
 
};
```

- MainMenuUI.cpp

``` cpp
// Copyright 1998-2015 Epic Games, Inc. All Rights Reserved.
 
 
#include "SlateTutorials.h"
#include "MainMenuUI.h"
#include "GlobalMenuStyle.h" 
#include "MenuStyles.h" 
 
 
BEGIN_SLATE_FUNCTION_BUILD_OPTIMIZATION
void SMainMenuUI::Construct(const FArguments& args)
{
    MainMenuHUD = args._MainMenuHUD;
    MenuStyle = &FMenuStyles::Get().GetWidgetStyle<FGlobalStyle>("Global");
 
    ChildSlot
    [
        SNew(SOverlay)
        + SOverlay::Slot()
        .HAlign(HAlign_Center)
        .VAlign(VAlign_Top)
        [
            SNew(STextBlock)
            .TextStyle(&MenuStyle->MenuTitleStyle)
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
                .ButtonStyle(&MenuStyle->MenuButtonStyle)
                .TextStyle(&MenuStyle->MenuButtonTextStyle)
                .Text(FText::FromString("Play Game!"))
                .OnClicked(this, &SMainMenuUI::PlayGameClicked)
            ]
            + SVerticalBox::Slot()
            [
                SNew(SButton)
                .ButtonStyle(&MenuStyle->MenuButtonStyle)
                .TextStyle(&MenuStyle->MenuButtonTextStyle)
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

----------

## **<font color=#191970 size=5>总结</font>** ##

瞧！您的菜单现在已设置样式！这里我们做了什么的细节：<br>
首先，在绑定我们的<font color=#dd1144 size=3>MainMenuHUD</font>后，实际上通过我们的<font color=#dd1144 size=3>FMenuStyles</font>类在布局前面加载<font color=#dd1144 size=3>Slate Widget</font>样式。<br>
接下来，我们调整游戏标题的STextBlock，以添加对<font color=#dd1144 size=3>TextStyle()</font>的调用，并往其中传递我们的标题文本样式的地址。<br>这与Slate的属性调整完全相同！对于我们的两个按钮，我们实际分配两种样式。 首先，我们分配我们的按钮样式，然后我们分配文本样式 - 没有太多担心这里，对吧？你现在有一个菜单，有一些风格化的按钮！<br>
编译并开始你的游戏，然后测试你的主菜单！

![示例图片](https://d26ilriwvtzlb.cloudfront.net/9/99/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82_2015-03-17_17.49.48.png)