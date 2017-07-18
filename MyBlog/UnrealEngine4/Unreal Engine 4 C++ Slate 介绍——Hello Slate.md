[TOC]

# **Unreal Engine 4 C++ Slate 介绍——Hello Slate**#

好记性不如烂笔头啊，还是记录一下!

----------

<font color=#dd1144 size=4>Slate</font>是UE4的用户界面系统，UE4编辑器的大部分界面都是由<font color=#dd1144 size=3>Slate</font>构建的。同时，在编辑器中使用的<font color=#dd1144 size=3>UMG</font>也是在<font color=#dd1144 size=3>Slate</font>的基础上封装的。

本教程需要对C++和事件驱动系统有一定的了解。本教程会对于如何添加HUD并且渲染一个Slate窗口做一个基本的介绍，Slate窗口会显示一个消息为“Hello, Slate!”的文本框。除了这些，需要几个步骤去设置工程才能处理Slate窗口。这里做的一些事情对有经验的人来说显得有些直白，但是这些事情对那些刚开始接触的人并不是很明白，所以还是有必要讲一下。

## **<font color=#191970 size=5>创建一个工程</font> ** ##

创建一个工程，并且创建以下文件：

- <font color=#dd1144 size=3>StandardHUD.h</font>
- <font color=#dd1144 size=3>StandardHUD.cpp</font>
- <font color=#dd1144 size=3>StandardSlateWidget.h</font>
- <font color=#dd1144 size=3>StandardSlateWidget.cpp</font>

<font color=#dd1144 size=3>StandardHUD</font>类需要继承自<font color=#dd1144 size=3>HUD</font>类，<font color=#dd1144 size=3>StandardSlateWidget</font>类需要继承自<font color=#dd1144 size=3>CompoundWidget</font>类（这个列表里选不到，创建个空白的C++类，后续有修改方法）

## **<font color=#191970 size=5>修改工程配置增加Slate的依赖项</font> ** ##

打开你创建的项目的配置文件（<font color=#dd1144 size=3><你的项目名称></font>.Build.cs）,取消以下行的注释，并修改成这样：

``` cs
PrivateDependencyModuleNames.AddRange(
    new string[] {
        "Slate",
        "SlateCore"
    }
);
```

保存关闭后，需要重新编译你的工程。

## **<font color=#191970 size=5>编写代码</font> ** ##

现在我们的工程里有4个文件，现在都是空的，下面是每个文件的代码。<font color=#dd1144 size=3>StandardHUD</font>类基本上和<font color=#dd1144 size=3>HUD</font>类是一样的。<font color=#dd1144 size=3>StandardSlateWidget</font>是我们自定的控件，这是我们要显示文本的地方。

现在我们的代码文件为：

- <font color=#dd1144 size=3>StandardHUD.h</font>
- <font color=#dd1144 size=3>StandardHUD.cpp</font>
- <font color=#dd1144 size=3>StandardSlateWidget.h</font>
- <font color=#dd1144 size=3>StandardSlateWidget.cpp</font>

他们的代码分别为：

- <font color=#dd1144 size=3>StandardHUD.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.
#pragma once
#include "GameFramework/HUD.h"
#include "StandardHUD.generated.h"
 
class SStandardSlateWidget;
 
UCLASS()
class AStandardHUD : public AHUD
{
    GENERATED_BODY()
public:
    AStandardHUD();
 
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    /////Reference to an SCompoundWidget, TSharedPtr adds to the refcount of MyUIWidget
    /////MyUIWidget will not self-destruct as long as refcount > 0
    /////MyUIWidget refcount will be (refcout-1) if HUD is destroyed.
    TSharedPtr<SStandardSlateWidget> MyUIWidget;
 
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    /////Called as soon as game starts, create SCompoundWidget and give Viewport access
    void BeginPlay();
};
```

- <font color=#dd1144 size=3>StandardHUD.cpp</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.
 
#include "HelloSlate.h"
#include "StandardSlateWidget.h"
#include "StandardHUD.h"
 
AStandardHUD::AStandardHUD()
{
 
}
 
void AStandardHUD::BeginPlay()
{
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    /////So far only TSharedPtr<SMyUIWidget> has been created, now create the actual object.
    /////Create a SMyUIWidget on heap, our MyUIWidget shared pointer provides handle to object
    /////Widget will not self-destruct unless the HUD's SharedPtr (and all other SharedPtrs) destruct first.
    MyUIWidget = SNew(SStandardSlateWidget).OwnerHUD(this);
 
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    /////Pass our viewport a weak ptr to our widget
    /////Viewport's weak ptr will not give Viewport ownership of Widget
    GEngine->GameViewport->AddViewportWidgetContent(SNew(SWeakWidget).PossiblyNullContent(MyUIWidget.ToSharedRef()));
 
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    /////Set widget's properties as visible (sets child widget's properties recursively)
    MyUIWidget->SetVisibility(EVisibility::Visible);
}
```

- <font color=#dd1144 size=3>StandardSlateWidget.h</font>

``` cpp
//Copyright 1998-2014 Epic Games, Inc. All Rights Reserved.
 
#pragma once
#include "StandardHUD.h"
#include "SlateBasics.h"
 
class SStandardSlateWidget : public SCompoundWidget
{
    SLATE_BEGIN_ARGS(SStandardSlateWidget)
    {
    }
 
    /* 参照下面的OwnerHUD的声明 */
    SLATE_ARGUMENT(TWeakObjectPtr<class AStandardHUD>, OwnerHUD)
 
    SLATE_END_ARGS()
 
public:

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    ///// 每一个控件都必须要有这个函数
    ///// 构建控件及其子控件
    void Construct(const FArguments& InArgs);
 
private:
    
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    ///// 指向控件的持有者Hud
    ///// 使用弱引用持有HUD的指针，因为HUD是使用强引用来持有Widget的。
    ///// 如果双方都为强引用的话将会导致解构时形成循环引用并引发内存泄露
    TWeakObjectPtr<class AStandardHUD> OwnerHUD;
};
```

- <font color=#dd1144 size=3>StandardSlateWidget.cpp</font>

``` cpp
#include "HelloSlate.h"
#include "StandardSlateWidget.h"
#include "StandardHUD.h"
 
#define LOCTEXT_NAMESPACE "SStandardSlateWidget"
 
void SStandardSlateWidget::Construct(const FArguments& InArgs)
{
    OwnerHUD = InArgs._OwnerHUD;
 
    ////////////////////////////////////////////////////////////////////////////////////////////////////
    /////If the code below doesn't look like C++ to you it's because it (sort-of) isn't,
    /////Slate makes extensive use of the C++ Prerocessor(macros) and operator overloading,
    /////Epic is trying to make our lives easier, look-up the macro/operator definitions to see why.
    ChildSlot
    .VAlign(VAlign_Fill)
    .HAlign(HAlign_Fill)
    [
        SNew(SOverlay)
        +SOverlay::Slot()
        .VAlign(VAlign_Top)
        .HAlign(HAlign_Center)
        [
            SNew(STextBlock)
            .ShadowColorAndOpacity(FLinearColor::Black)
            .ColorAndOpacity(FLinearColor::Red)
            .ShadowOffset(FIntPoint(-1, 1))
                        .Font(FSlateFontInfo("Veranda", 16))
            .Text(LOCTEXT("HelloSlate", "Hello, Slate!"))
        ]
    ];
}
 
#undef LOCTEXT_NAMESPACE
```

现在你可以重新编译下你的工程，你会发现屏幕中间并没有显示任何东西，因为在游戏模式中并没有把我们的HUD设置为默认的。

在你的游戏模式的文件中（*GameMode.cpp）中，在构造函数中添加这行代码：

``` cpp
HUDClass = AStandardHUD::StaticClass();
```

再次编译会发现屏幕上有了一行文字：

![示例](https://d3ar1piqh1oeli.cloudfront.net/3/38/Hello_slate.png/400px-Hello_slate.png)

