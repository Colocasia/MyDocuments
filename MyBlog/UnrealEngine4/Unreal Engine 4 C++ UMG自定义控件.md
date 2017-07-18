[TOC]

# **Unreal Engine 4 C++ UMG自定义控件**#

好记性不如烂笔头啊，还是记录一下!

----------

如果你觉得Unreal Engine里面的控件没有达到你的需求，你需要添加自定控件。

----------

## **<font color=#191970 size=5>创建Slate控件</font> ** ##

如果你还不了解如何创建一个<font color=#dd1144 size=3>Slate</font>控件，请先阅读：

>[Unreal Engine 4 C++ Slate 介绍——用C++和Slate创建菜单（一）](http://blog.csdn.net/qq_20309931/article/details/53289032)<br>
>[Unreal Engine 4 C++ Slate 介绍——用C++和Slate创建菜单（二）](http://blog.csdn.net/qq_20309931/article/details/53303332)<br>
>[Unreal Engine 4 C++ Slate 介绍——用C++和Slate创建菜单（三）](http://blog.csdn.net/qq_20309931/article/details/53311238)<br>

这里用我自己写的作为参考：

- <font color=#dd1144 size=3>SProgressBarEx.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "UI/Slate/ProgressBarExStyle.h"

/**
 *  SProgressBarEx Fill Type 
 */
UENUM(BlueprintType)
namespace EProgressBarExFillType
{
    enum Type
    {
        // will fill up from the left side to the right
        LeftToRight,
        // will fill up from the right side to the left side
        RightToLeft,
        // will fill up from the center to the outer edges
        FillFromCenter,
        // will fill up from the top to the the bottom
        TopToBottom,
        // will fill up from the bottom to the the top
        BottomToTop,
    };
}

/** A progress extend bar widget.*/
class TESTMOBILE_API SProgressBarEx : public SLeafWidget
{

public:
    SLATE_BEGIN_ARGS(SProgressBarEx)
        : _Style( nullptr )
        , _BarFillType(EProgressBarExFillType::LeftToRight)
        , _Percent( TOptional<float>() )
        , _FillColorAndOpacity( FLinearColor::White )
        , _BorderPadding( FVector2D(1,0) )
        , _BackgroundImage(&FCoreStyle::Get().GetWidgetStyle<FProgressBarStyle>("ProgressBar").BackgroundImage)
        , _IncreaseImage(&FCoreStyle::Get().GetWidgetStyle<FProgressBarStyle>("ProgressBar").FillImage)
        , _DecreaseImage(&FCoreStyle::Get().GetWidgetStyle<FProgressBarStyle>("ProgressBar").FillImage)
        , _FillImage(&FCoreStyle::Get().GetWidgetStyle<FProgressBarStyle>("ProgressBar").FillImage)
        , _MarqueeImage(&FCoreStyle::Get().GetWidgetStyle<FProgressBarStyle>("ProgressBar").MarqueeImage)
        , _RefreshRate(2.0f)
        {}

        /** Style used for the progress bar */
        SLATE_STYLE_ARGUMENT( FProgressBarExStyle, Style )

        /** Defines if this progress bar fills Left to right or right to left*/
        SLATE_ARGUMENT( EProgressBarExFillType::Type, BarFillType )

        /** Used to determine the fill position of the progress bar ranging 0..1 */
        SLATE_ATTRIBUTE( TOptional<float>, Percent )

        /** Fill Color and Opacity */
        SLATE_ATTRIBUTE( FSlateColor, FillColorAndOpacity )

        /** Border Padding around fill bar */
        SLATE_ATTRIBUTE( FVector2D, BorderPadding )
    
        /** The brush to use as the background of the progress bar */
        SLATE_ARGUMENT(const FSlateBrush*, BackgroundImage)

        /** The brush to use as the increase image of the progress bar */
        SLATE_ARGUMENT(const FSlateBrush*, IncreaseImage)

        /** The brush to use as the decrease image of the progress bar */
        SLATE_ARGUMENT(const FSlateBrush*, DecreaseImage)
    
        /** The brush to use as the fill image */
        SLATE_ARGUMENT(const FSlateBrush*, FillImage)
    
        /** The brush to use as the marquee image */
        SLATE_ARGUMENT(const FSlateBrush*, MarqueeImage)

        /** Rate at which this widget is ticked when sleeping in seconds */
        SLATE_ARGUMENT(float, RefreshRate)

    SLATE_END_ARGS()

    /**
     * Construct the widget
     * 
     * @param InArgs   A declaration from which to construct the widget
     */
    void Construct(const FArguments& InArgs);

    /** Paint Widget */
    virtual int32 OnPaint( const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyClippingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled ) const override;
    virtual FVector2D ComputeDesiredSize(float) const override;
    virtual bool ComputeVolatility() const override;

    /** See attribute Percent */
    void SetPercent(TAttribute< TOptional<float> > InPercent);

    /** See attribute Style */
    void SetStyle(const FProgressBarExStyle* InStyle);

    /** See attribute BarFillType */
    void SetBarFillType(EProgressBarExFillType::Type InBarFillType);
    
    /** See attribute SetFillColorAndOpacity */
    void SetFillColorAndOpacity(TAttribute< FSlateColor > InFillColorAndOpacity);
    
    /** See attribute BorderPadding */
    void SetBorderPadding(TAttribute< FVector2D > InBorderPadding);
    
    /** See attribute BackgroundImage */
    void SetBackgroundImage(const FSlateBrush* InBackgroundImage);

    /** See attribute IncreaseImage */
    void SetIncreaseImage(const FSlateBrush* InIncreaseImage);

    /** See attribute IncreaseImage */
    void SetDecreaseImage(const FSlateBrush* InDecreaseImage);
    
    /** See attribute FillImage */
    void SetFillImage(const FSlateBrush* InFillImage);
    
    /** See attribute MarqueeImage */
    void SetMarqueeImage(const FSlateBrush* InMarqueeImage);

private:

    /** Controls the speed at which the widget is ticked when in slate sleep mode */
    void SetActiveTimerTickRate(float TickRate);

    /** Widgets active tick */
    EActiveTimerReturnType ActiveTick(double InCurrentTime, float InDeltaTime);

    /** Gets the current background image. */
    const FSlateBrush* GetBackgroundImage() const;

    /** Gets the current increase image */
    const FSlateBrush* GetIncreaseImage() const;

    /** Gets the current decrease image */
    const FSlateBrush* GetDecreaseImage() const;

    /** Gets the current fill image */
    const FSlateBrush* GetFillImage() const;

    /** Gets the current marquee image */
    const FSlateBrush* GetMarqueeImage() const;


private:

    /** The style of the progress bar */
    const FProgressBarExStyle* Style;

    /** The text displayed over the progress bar */
    TAttribute< TOptional<float> > CurrentPercent;
    TAttribute< TOptional<float> > TargetPercent;
    TAttribute< TOptional<float> > MiddlePercent;

    /** The fill type for the progress bar */
    EProgressBarExFillType::Type BarFillType;

    /** Background image to use for the progress bar */
    const FSlateBrush* BackgroundImage;

    /** Increase image to use for the progress bar */
    const FSlateBrush* IncreaseImage;

    /** Decrease image to use for the progress bar */
    const FSlateBrush* DecreaseImage;

    /** Foreground image to use for the progress bar */
    const FSlateBrush* FillImage;

    /** Image to use for marquee mode */
    const FSlateBrush* MarqueeImage;

    /** Value to drive progress bar animation */
    float SpeedRate;

    /** Fill Color and Opacity */
    TAttribute<FSlateColor> FillColorAndOpacity;

    /** Border Padding */
    TAttribute<FVector2D> BorderPadding;

    /** Value to drive progress bar animation */
    float MarqueeOffset;

    /** Reference to the widgets current active timer */
    TWeakPtr<FActiveTimerHandle> ActiveTimerHandle;

    /** Rate at which the widget is currently ticked when slate sleep mode is active */
    float CurrentTickRate;

    /** The slowest that this widget can tick when in slate sleep mode */
    float MinimumTickRate;
};

```

- <font color=#dd1144 size=3>SProgressBarEx.cpp</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#include "TestMobile.h"
#include "SProgressBarEx.h"

void SProgressBarEx::Construct(const FArguments& InArgs)
{
    check(InArgs._BackgroundImage);
    check(InArgs._IncreaseImage);
    check(InArgs._DecreaseImage);
    check(InArgs._FillImage);
    check(InArgs._MarqueeImage);


    MarqueeOffset = 0.0f;
    Style = InArgs._Style;

    
    TAttribute< TOptional<float> > Percent = InArgs._Percent.Get().IsSet() ? InArgs._Percent : 0.5f;
    SetPercent(Percent);

    SpeedRate = 30.0f;

    BarFillType = InArgs._BarFillType;
    BackgroundImage = InArgs._BackgroundImage;
    IncreaseImage = InArgs._IncreaseImage;
    DecreaseImage = InArgs._DecreaseImage;
    FillImage = InArgs._FillImage;
    MarqueeImage = InArgs._MarqueeImage;

    FillColorAndOpacity = InArgs._FillColorAndOpacity;
    BorderPadding = InArgs._BorderPadding;

    CurrentTickRate = 0.0f;
    MinimumTickRate = InArgs._RefreshRate;

    ActiveTimerHandle = RegisterActiveTimer(CurrentTickRate, FWidgetActiveTimerDelegate::CreateSP(this, &SProgressBarEx::ActiveTick));
}


void SProgressBarEx::SetPercent(TAttribute< TOptional<float> > InPercent)
{
    if (!TargetPercent.IdenticalTo(InPercent))
    {
        CurrentPercent = MiddlePercent = TargetPercent = InPercent;
        Invalidate(EInvalidateWidget::LayoutAndVolatility);
    }
}


void SProgressBarEx::SetStyle(const FProgressBarExStyle* InStyle)
{
    Style = InStyle;

    if (Style == nullptr)
    {
        FArguments Defaults;
        Style = Defaults._Style;
    }

    check(Style);

    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetBarFillType(EProgressBarExFillType::Type InBarFillType)
{
    BarFillType = InBarFillType;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetFillColorAndOpacity(TAttribute< FSlateColor > InFillColorAndOpacity)
{
    FillColorAndOpacity = InFillColorAndOpacity;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetBorderPadding(TAttribute< FVector2D > InBorderPadding)
{
    BorderPadding = InBorderPadding;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetBackgroundImage(const FSlateBrush* InBackgroundImage)
{
    BackgroundImage = InBackgroundImage;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetIncreaseImage(const FSlateBrush* InIncreaseImage)
{
    IncreaseImage = InIncreaseImage;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetDecreaseImage(const FSlateBrush* InDecreaseImage)
{
    DecreaseImage = InDecreaseImage;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetFillImage(const FSlateBrush* InFillImage)
{
    FillImage = InFillImage;
    Invalidate(EInvalidateWidget::Layout);
}


void SProgressBarEx::SetMarqueeImage(const FSlateBrush* InMarqueeImage)
{
    MarqueeImage = InMarqueeImage;
    Invalidate(EInvalidateWidget::Layout);
}


const FSlateBrush* SProgressBarEx::GetBackgroundImage() const
{
    return &Style->BackgroundImage ? &Style->BackgroundImage : BackgroundImage;
}


const FSlateBrush* SProgressBarEx::GetIncreaseImage() const
{
    return &Style->IncreaseImage ? &Style->IncreaseImage : IncreaseImage;
}


const FSlateBrush* SProgressBarEx::GetDecreaseImage() const
{
    return &Style->DecreaseImage ? &Style->DecreaseImage : DecreaseImage;
}


const FSlateBrush* SProgressBarEx::GetFillImage() const
{
    return &Style->FillImage ? &Style->FillImage : FillImage;
}


const FSlateBrush* SProgressBarEx::GetMarqueeImage() const
{
    return &Style->MarqueeImage ? &Style->MarqueeImage : MarqueeImage;
}


int32 SProgressBarEx::OnPaint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyClippingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const
{
    // Used to track the layer ID we will return.
    int32 RetLayerId = LayerId;

    bool bEnabled = ShouldBeEnabled( bParentEnabled );
    const ESlateDrawEffect::Type DrawEffects = bEnabled ? ESlateDrawEffect::None : ESlateDrawEffect::DisabledEffect;

    const FSlateBrush* CurrentFillImage = GetFillImage();
    const FLinearColor FillColorAndOpacitySRGB(InWidgetStyle.GetColorAndOpacityTint() * FillColorAndOpacity.Get().GetColor(InWidgetStyle) * CurrentFillImage->GetTint(InWidgetStyle));
    const FLinearColor ColorAndOpacitySRGB = InWidgetStyle.GetColorAndOpacityTint();

    // Paint inside the border only. 
    // Pre-snap the clipping rect to try and reduce common jitter, since the padding is typically only a single pixel.
    FSlateRect SnappedClippingRect = FSlateRect(FMath::RoundToInt(MyClippingRect.Left), FMath::RoundToInt(MyClippingRect.Top), FMath::RoundToInt(MyClippingRect.Right), FMath::RoundToInt(MyClippingRect.Bottom));
    const FSlateRect ForegroundClippingRect = SnappedClippingRect.InsetBy(FMargin(BorderPadding.Get().X, BorderPadding.Get().Y));

    // Paint background image
    const FSlateBrush* CurrentBackgroundImage = GetBackgroundImage();
    FSlateDrawElement::MakeBox(
        OutDrawElements,
        RetLayerId++,
        AllottedGeometry.ToPaintGeometry(),
        CurrentBackgroundImage,
        SnappedClippingRect,
        DrawEffects,
        InWidgetStyle.GetColorAndOpacityTint() * CurrentBackgroundImage->GetTint( InWidgetStyle )
    );

    TOptional<float> TargetPercentFraction = TargetPercent.Get();
    TOptional<float> MiddlePercentFraction = MiddlePercent.Get();
    TOptional<float> CurrentPercentFraction = CurrentPercent.Get();

    if ( TargetPercentFraction.IsSet() && MiddlePercentFraction.IsSet() && CurrentPercentFraction.IsSet() )
    {
        // Get percent value
        float TargetPercentValue = FMath::Clamp(TargetPercentFraction.GetValue(), 0.0f, 1.0f);
        float MiddlePercentValue = FMath::Clamp(MiddlePercentFraction.GetValue(), 0.0f, 1.0f);
        float CurrentPercentValue = FMath::Clamp(CurrentPercentFraction.GetValue(), 0.0f, 1.0f);

        // Get paint order
        const FSlateBrush* TopBrush;
        const FSlateBrush* CenterBrush;
        const FSlateBrush* BottomBrush;
        float TopValue;
        float CenterValue;
        float BottomValue;
       

        TopBrush = GetFillImage();
        TopValue = MiddlePercentValue;
        CenterBrush = GetIncreaseImage();
        CenterValue = TargetPercentValue;
        BottomBrush = GetDecreaseImage();
        BottomValue = CurrentPercentValue;


        

        switch (BarFillType)
        {
        case EProgressBarExFillType::RightToLeft:
            break;
        case EProgressBarExFillType::FillFromCenter:
            break;
        case EProgressBarExFillType::TopToBottom:
            break;
        case EProgressBarExFillType::BottomToTop:
            break;
        case EProgressBarExFillType::LeftToRight:
        default:
            // Draw bottom layer
            FSlateRect ClippedAllotedGeometry = FSlateRect(AllottedGeometry.AbsolutePosition, AllottedGeometry.AbsolutePosition + AllottedGeometry.Size * AllottedGeometry.Scale);
            ClippedAllotedGeometry.Right = ClippedAllotedGeometry.Left + ClippedAllotedGeometry.GetSize().X * BottomValue;
            FSlateDrawElement::MakeBox(
                OutDrawElements,
                RetLayerId++,
                AllottedGeometry.ToPaintGeometry(
                    FVector2D::ZeroVector,
                    FVector2D( AllottedGeometry.Size.X, AllottedGeometry.Size.Y )),
                BottomBrush,
                ForegroundClippingRect.IntersectionWith(ClippedAllotedGeometry),
                DrawEffects,
                FillColorAndOpacitySRGB
                );

            // Draw center layer
            ClippedAllotedGeometry =  FSlateRect(AllottedGeometry.AbsolutePosition, AllottedGeometry.AbsolutePosition + AllottedGeometry.Size * AllottedGeometry.Scale);
            ClippedAllotedGeometry.Right = ClippedAllotedGeometry.Left + ClippedAllotedGeometry.GetSize().X * CenterValue;
            FSlateDrawElement::MakeBox(
                OutDrawElements,
                RetLayerId++,
                AllottedGeometry.ToPaintGeometry(
                    FVector2D::ZeroVector,
                    FVector2D( AllottedGeometry.Size.X, AllottedGeometry.Size.Y )),
                CenterBrush,
                ForegroundClippingRect.IntersectionWith(ClippedAllotedGeometry),
                DrawEffects,
                FillColorAndOpacitySRGB
                );

            // Draw top layer
            ClippedAllotedGeometry =  FSlateRect(AllottedGeometry.AbsolutePosition, AllottedGeometry.AbsolutePosition + AllottedGeometry.Size * AllottedGeometry.Scale);
            ClippedAllotedGeometry.Right = ClippedAllotedGeometry.Left + ClippedAllotedGeometry.GetSize().X * TopValue;
            FSlateDrawElement::MakeBox(
                OutDrawElements,
                RetLayerId++,
                AllottedGeometry.ToPaintGeometry(
                    FVector2D::ZeroVector,
                    FVector2D( AllottedGeometry.Size.X, AllottedGeometry.Size.Y )),
                TopBrush,
                ForegroundClippingRect.IntersectionWith(ClippedAllotedGeometry),
                DrawEffects,
                FillColorAndOpacitySRGB
                );
            break;
        }
    }
    else
    {

    }

    return RetLayerId - 1;
}


FVector2D SProgressBarEx::ComputeDesiredSize(float) const
{
    return GetMarqueeImage()->ImageSize;
}


bool SProgressBarEx::ComputeVolatility() const
{
    return SLeafWidget::ComputeVolatility() || TargetPercent.IsBound();
}


void SProgressBarEx::SetActiveTimerTickRate(float TickRate)
{
    if (CurrentTickRate != TickRate || !ActiveTimerHandle.IsValid())
    {
        CurrentTickRate = TickRate;

        TSharedPtr<FActiveTimerHandle> SharedActiveTimerHandle = ActiveTimerHandle.Pin();
        if (SharedActiveTimerHandle.IsValid())
        {
            UnRegisterActiveTimer(SharedActiveTimerHandle.ToSharedRef());
        }

        ActiveTimerHandle = RegisterActiveTimer(TickRate, FWidgetActiveTimerDelegate::CreateSP(this, &SProgressBarEx::ActiveTick));
    }
}


EActiveTimerReturnType SProgressBarEx::ActiveTick(double InCurrentTime, float InDeltaTime)
{
    MarqueeOffset = InCurrentTime - FMath::FloorToDouble(InCurrentTime);

    TOptional<float> TargetPercentFraction = TargetPercent.Get();
    if (TargetPercentFraction.IsSet())
    {
        SetActiveTimerTickRate(MinimumTickRate);
    }
    else
    {
        SetActiveTimerTickRate(0.0f);
    }

    return EActiveTimerReturnType::Continue;
}
```

这个是一个多层进度条的Slate控件，紧接着要为他写个Slate样式。

- <font color=#dd1144 size=3>ProgressBarExStyle.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "SlateWidgetStyleContainerBase.h"
#include "SlateWidgetStyle.h"
#include "SlateBasics.h"
#include "ProgressBarExStyle.generated.h"


/**
 * Represents the appearance of an SProgressBarEx
 */
USTRUCT(BlueprintType)
struct TESTMOBILE_API FProgressBarExStyle : public FSlateWidgetStyle
{
    GENERATED_USTRUCT_BODY()

    FProgressBarExStyle();

    virtual ~FProgressBarExStyle() {}

    virtual void GetResources( TArray< const FSlateBrush* >& OutBrushes ) const override;

    static const FName TypeName;
    virtual const FName GetTypeName() const override { return TypeName; };

    static const FProgressBarExStyle& GetDefault();

    /** Background image to use for the progress bar */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Appearance)
    FSlateBrush BackgroundImage;
    FProgressBarExStyle& SetBackgroundImage( const FSlateBrush& InBackgroundImage ){ BackgroundImage = InBackgroundImage; return *this; }

    /** Increase image to use for the progress bar */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Appearance)
    FSlateBrush IncreaseImage;
    FProgressBarExStyle& SetIncreaseImage( const FSlateBrush& InIncreaseImage ){ IncreaseImage = InIncreaseImage; return *this; }

    /** Decrease image to use for the progress bar */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Appearance)
    FSlateBrush DecreaseImage;
    FProgressBarExStyle& SetDecreaseImage( const FSlateBrush& InDecreaseImage ){ DecreaseImage = InDecreaseImage; return *this; }

    /** Foreground image to use for the progress bar */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Appearance)
    FSlateBrush FillImage;
    FProgressBarExStyle& SetFillImage( const FSlateBrush& InFillImage ){ FillImage = InFillImage; return *this; }

    /** Image to use for marquee mode */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Appearance)
    FSlateBrush MarqueeImage;
    FProgressBarExStyle& SetMarqueeImage( const FSlateBrush& InMarqueeImage ){ MarqueeImage = InMarqueeImage; return *this; }
};
```

- <font color=#dd1144 size=3>ProgressBarExStyle.cpp</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#include "TestMobile.h"
#include "ProgressBarExStyle.h"

FProgressBarExStyle::FProgressBarExStyle()
{
}

void FProgressBarExStyle::GetResources( TArray< const FSlateBrush* >& OutBrushes ) const
{
    OutBrushes.Add( &BackgroundImage );
    OutBrushes.Add( &IncreaseImage );
    OutBrushes.Add( &DecreaseImage );
    OutBrushes.Add( &FillImage );
    OutBrushes.Add( &MarqueeImage );
}

const FName FProgressBarExStyle::TypeName( TEXT("FProgressBarExStyle") );

const FProgressBarExStyle& FProgressBarExStyle::GetDefault()
{
    static FProgressBarExStyle Default;
    return Default;
}
```

到此我们就结束了Slate控件的编写，只是写了个基本的多层进度条。

----------

## **<font color=#191970 size=5>添加到UMG编辑器中</font> ** ##

首先需要给样式加上一个基础容器，并且要反射给UMG的编辑器

- <font color=#dd1144 size=3>ProgressExWidgetStyle.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "Styling/SlateWidgetStyleContainerBase.h"
#include "UI/Slate/ProgressBarExStyle.h"
#include "ProgressExWidgetStyle.generated.h"

/**
 * 
 */
UCLASS(BlueprintType, hidecategories=Object, MinimalAPI)
class UProgressExWidgetStyle : public USlateWidgetStyleContainerBase
{
    GENERATED_BODY()

public:

    /** The actual data describing the button's appearance. */
    UPROPERTY(Category="Style", EditAnywhere, BlueprintReadWrite, meta=(ShowOnlyInnerProperties))
    FProgressBarExStyle ProgressBarExStyle;
    
    virtual const struct FSlateWidgetStyle* const GetStyle() const override
    {
        return static_cast< const struct FSlateWidgetStyle* >( &ProgressBarExStyle );
    }
};
```

然后要让编辑器能看到我们的Slate控件：

- <font color=#dd1144 size=3>ProgressBarEx.h</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "Widget.h"
#include "ProgressExWidgetStyle.h"
#include "UI/Slate/SProgressBarEx.h"
#include "ProgressBarEx.generated.h"

class USlateBrushAsset;

/**
 *  ProgressBarEx
 */
UCLASS()
class TESTMOBILE_API UProgressBarEx : public UWidget
{
    GENERATED_BODY()

public:

    /** The progress bar ex default setting */
    UProgressBarEx(const FObjectInitializer& ObjectInitializer = FObjectInitializer::Get());
    
    /** The progress bar ex style */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Style", meta=( DisplayName="Style" ))
    FProgressBarExStyle WidgetStyle;

    /** Style used for the progress bar */
    UPROPERTY()
    USlateWidgetStyleAsset* Style_DEPRECATED;

    /** The brush to use as the background of the progress bar */
    UPROPERTY()
    USlateBrushAsset* BackgroundImage_DEPRECATED;

    /** The brush to use as the increase image of the progress bar */
    UPROPERTY()
    USlateBrushAsset* IncreaseImage_DEPRECATED;

    /** The brush to use as the decrease image of the progress bar */
    UPROPERTY()
    USlateBrushAsset* DecreaseImage_DEPRECATED;

    /** The brush to use as the fill image */
    UPROPERTY()
    USlateBrushAsset* FillImage_DEPRECATED;

    /** The brush to use as the marquee image */
    UPROPERTY()
    USlateBrushAsset* MarqueeImage_DEPRECATED;

    /** Used to determine the fill position of the progress bar ranging 0..1 */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=Progress, meta=( UIMin = "0", UIMax = "1" ))
    float Percent;

    /** Defines if this progress bar fills Left to right or right to left */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=Progress)
    TEnumAsByte<EProgressBarExFillType::Type> BarFillType;
    
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=Progress)
    bool bIsMarquee;

    /** A bindable delegate to allow logic to drive the text of the widget */
    UPROPERTY()
    FGetFloat PercentDelegate;

    /** Fill Color and Opacity */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=Appearance)
    FLinearColor FillColorAndOpacity;

    /** */
    UPROPERTY()
    FGetLinearColor FillColorAndOpacityDelegate;
    
public:
    
    /** Sets the current value of the ProgressBar. */
    UFUNCTION(BlueprintCallable, Category="Progress")
    void SetPercent(float InPercent);

    /** Sets the fill color of the progress bar. */
    UFUNCTION(BlueprintCallable, Category="Progress")
    void SetFillColorAndOpacity(FLinearColor InColor);

    /** Sets the progress bar to show as a marquee. */
    UFUNCTION(BlueprintCallable, Category="Progress")
    void SetIsMarquee(bool InbIsMarquee);

    //TODO UMG Add Set BarFillType.

public:

    //~ Begin UWidget Interface
    virtual void SynchronizeProperties() override;
    //~ End UWidget Interface

    //~ Begin UVisual Interface
    virtual void ReleaseSlateResources(bool bReleaseChildren) override;
    //~ End UVisual Interface

    //~ Begin UObject Interface
    virtual void PostLoad() override;
    //~ End UObject Interface

#if WITH_EDITOR

    //~ Begin UWidget Interface
    virtual const FText GetPaletteCategory() override;
    virtual void OnCreationFromPalette() override;
    //~ End UWidget Interface

#endif

protected:

    /** Native Slate Widget */
    TSharedPtr<SProgressBarEx> MyProgressBar;

    //~ Begin UWidget Interface
    virtual TSharedRef<SWidget> RebuildWidget() override;
    //~ End UWidget Interface
};

```

- <font color=#dd1144 size=3>ProgressBarEx.cpp</font>

``` cpp
// Fill out your copyright notice in the Description page of Project Settings.

#include "TestMobile.h"
#include "UMG.h"
#include "UMGStyle.h"
#include "Slate/SlateBrushAsset.h"
#include "ProgressBarEx.h"

#define LOCTEXT_NAMESPACE "UMG"

/////////////////////////////////////////////////////
// UProgressBarEx

UProgressBarEx::UProgressBarEx(const FObjectInitializer& ObjectInitializer /*= FObjectInitializer::Get()*/)
    : Super(ObjectInitializer)
{
    // SProgressBarEx::FArguments SlateDefaults;
    // WidgetStyle = *SlateDefaults._Style;
    // WidgetStyle = FMenuStyles::Get().GetWidgetStyle<FProgressBarExStyle>("HPBarStyle");
    // WidgetStyle.FillImage.TintColor = FLinearColor::White;
    
    BarFillType = EProgressBarExFillType::LeftToRight;
    bIsMarquee = false;
    Percent = 0;
    FillColorAndOpacity = FLinearColor::White;
}


void UProgressBarEx::SetIsMarquee(bool InbIsMarquee)
{
    bIsMarquee = InbIsMarquee;
    if ( MyProgressBar.IsValid() )
    {
        MyProgressBar->SetPercent(bIsMarquee ? TOptional<float>() : Percent);
    }
}

void UProgressBarEx::SetFillColorAndOpacity(FLinearColor Color)
{
    FillColorAndOpacity = Color;
    if (MyProgressBar.IsValid())
    {
        MyProgressBar->SetFillColorAndOpacity(FillColorAndOpacity);
    }
}

void UProgressBarEx::SetPercent(float InPercent)
{
    Percent = InPercent;
    if ( MyProgressBar.IsValid() )
    {
        MyProgressBar->SetPercent(InPercent);
    }
}


void UProgressBarEx::SynchronizeProperties()
{
    Super::SynchronizeProperties();

    TAttribute< TOptional<float> > PercentBinding = OPTIONAL_BINDING_CONVERT(float, Percent, TOptional<float>, ConvertFloatToOptionalFloat);
    TAttribute<FSlateColor> FillColorAndOpacityBinding = OPTIONAL_BINDING(FSlateColor, FillColorAndOpacity);


    // MyProgressBar->SetStyle(&WidgetStyle);

    SProgressBarEx::FArguments SlateDefaults;
    MyProgressBar->SetBackgroundImage(SlateDefaults._BackgroundImage);
    MyProgressBar->SetIncreaseImage(SlateDefaults._IncreaseImage);
    MyProgressBar->SetDecreaseImage(SlateDefaults._DecreaseImage);
    MyProgressBar->SetFillImage(SlateDefaults._FillImage);
    MyProgressBar->SetMarqueeImage(SlateDefaults._MarqueeImage);

    MyProgressBar->SetStyle(&WidgetStyle);

    MyProgressBar->SetBarFillType(BarFillType);
    MyProgressBar->SetPercent(bIsMarquee ? TOptional<float>() : PercentBinding);
    MyProgressBar->SetFillColorAndOpacity(FillColorAndOpacityBinding);
}

void UProgressBarEx::PostLoad()
{
    Super::PostLoad();

    if ( GetLinkerUE4Version() < VER_UE4_DEPRECATE_UMG_STYLE_ASSETS )
    {
        if ( Style_DEPRECATED != nullptr )
        {
            const FProgressBarExStyle* StylePtr = Style_DEPRECATED->GetStyle<FProgressBarExStyle>();
            if ( StylePtr != nullptr )
            {
                WidgetStyle = *StylePtr;
            }

            Style_DEPRECATED = nullptr;
        }

        if ( BackgroundImage_DEPRECATED != nullptr )
        {
            WidgetStyle.BackgroundImage = BackgroundImage_DEPRECATED->Brush;
            BackgroundImage_DEPRECATED = nullptr;
        }

        if ( IncreaseImage_DEPRECATED != nullptr )
        {
            WidgetStyle.IncreaseImage = IncreaseImage_DEPRECATED->Brush;
            IncreaseImage_DEPRECATED = nullptr;
        }

        if ( DecreaseImage_DEPRECATED != nullptr )
        {
            WidgetStyle.DecreaseImage = DecreaseImage_DEPRECATED->Brush;
            DecreaseImage_DEPRECATED = nullptr;
        }

        if ( FillImage_DEPRECATED != nullptr )
        {
            WidgetStyle.FillImage = FillImage_DEPRECATED->Brush;
            FillImage_DEPRECATED = nullptr;
        }

        if ( MarqueeImage_DEPRECATED != nullptr )
        {
            WidgetStyle.MarqueeImage = MarqueeImage_DEPRECATED->Brush;
            MarqueeImage_DEPRECATED = nullptr;
        }
    }
}


void UProgressBarEx::ReleaseSlateResources(bool bReleaseChildren)
{
    Super::ReleaseSlateResources(bReleaseChildren);

    MyProgressBar.Reset();
}


TSharedRef<SWidget> UProgressBarEx::RebuildWidget()
{
    MyProgressBar = SNew(SProgressBarEx)
        .BorderPadding(FVector2D(0.0f, 0.0f));

    return MyProgressBar.ToSharedRef();
}


#if WITH_EDITOR

const FText UProgressBarEx::GetPaletteCategory()
{
    return LOCTEXT("Common", "Common");
}

void UProgressBarEx::OnCreationFromPalette()
{
    FillColorAndOpacity = FLinearColor(0, 0.5f, 1.0f);
}

#endif


/////////////////////////////////////////////////////

#undef LOCTEXT_NAMESPACE

```

其中<font color=#dd1144 size=3>RebuildWidget</font>和<font color=#dd1144 size=3>SynchronizeProperties</font>函数很重要。

编译代码后，打开UMG的编辑器，你会发现<font color=#dd1144 size=3>Common</font>组件下多了一个<font color=#dd1144 size=3>Progress Bar Ex</font>的控件：

![自定义Slate控件](http://img.blog.csdn.net/20161203142604033)
