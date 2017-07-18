[TOC]

# **Unreal Engine 4 C++ UMG血条及头顶信息**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>创建血条控件</font> ** ##

首先在内容里创建一个UI的控件蓝图

![界面控件](http://img.blog.csdn.net/20161125102434227)

接着在控件中的CanvasPanel中创建一个ProgressBar

![ProgressBar](http://img.blog.csdn.net/20161125102939579)

设置锚点为下图中的样子：

![设置锚点](http://img.blog.csdn.net/20161125104410207)

细节设置可以跟根据你的喜好来。这里有几个必须要弄的东西：

- Alignment 设置为<font color=#dd1144 size=3>X:0.5 Y:0.5</font>
- BackGround Image 是<font color=#dd1144 size=3>背景图片</font>，根据你的需求弄。
- Fill Image 是<font color=#dd1144 size=3>填充图片</font>，同样根据你的需求弄
- Bar Fill Type 一般是选<font color=#dd1144 size=3>Left To Right</font>

![细节设置](http://img.blog.csdn.net/20161125105534097)

血条控件就创建完成了。

----------

## **<font color=#191970 size=5>使用编辑器添加血条</font> ** ##

我们先来介绍第一种方式，用编辑器添加血条。

往角色上添加一个新的WidgetComponent：

![添加WidgetComponent](http://img.blog.csdn.net/20161125112349537)

设置WidgetComponent如下图所示：

![设置WidgetComponent](http://img.blog.csdn.net/20161125112931523)

这里也有几个参数要讲一下：

- Space 有两种方式<font color=#dd1144 size=3>World</font>和<font color=#dd1144 size=3>Screen</font>
 > <font color=#dd1144 size=3>World</font>方式是绘制在场景中，会被物体遮挡，相当于一个场景的公告板。<br>
 > <font color=#dd1144 size=3>Screen</font>方式是绘制在屏幕上，不会被物体遮挡，并且一直面向摄像机。<br>

- Widget Class 就是选择你要渲染的控件，选择刚才你创建的空间蓝图。
- Draw Size 就是你要绘制的尺寸，根据你的需要调整。

拖动WidgetComponent到角色头顶位置，

调整完后我们来测试一下，就会发现血条正常显示在了头顶：

![示例图片](http://img.blog.csdn.net/20161125114330279)

>**<font color=#DC143C size=4>蓝图绑定并更新血条这里我就不介绍了（其实我不会用蓝图绑定。。。），本文着重介绍C++部分</font>** 

## **<font color=#191970 size=5>使用C++添加血条</font> ** ##

我们在来看第二种方式，用C++添加血条

首先在你角色类中添加以下属性：

- <font color=#dd1144 size=3>MyCharacter.h</font>

``` cpp

// 如果使用GENERATED_BODY()宏
// 添加构造函数AMyCharacter(const FObjectInitializer& ObjectInitializer = FObjectInitializer::Get());
// 
// 在本文我直接使用GENERATED_UCLASS_BODY()宏
GENERATED_UCLASS_BODY()

// ...

// WidgetComponent
UPROPERTY(EditAnywhere, Category = WidgetComponent)
class UWidgetComponent* WidgetComponent;
```

关于<font color=#dd1144 size=3>GENERATED_UCLASS_BODY()</font>和<font color=#dd1144 size=3>GENERATED_BODY()</font>宏，详见[Unreal Eegine 4 C++ UCLASS构造函数易出错分析](http://blog.csdn.net/qq_20309931/article/details/52964391)<br>

我们在头文件添加了一个UWidgetComponent的指针，接着就该实现了，在构造函数：

- <font color=#dd1144 size=3>MyCharacter.cpp</font>

``` cpp
AMyCharacter::AMyCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{
    
    // ....你的代码

    // 初始化WidgetComponent
    WidgetComponent = CreateDefaultSubobject<UWidgetComponent>(TEXT("WidgetComponent"));
    WidgetComponent->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform);

    // 设置Widget Class
    UClass* Widget = LoadClass<UUserWidget>(NULL, TEXT("WidgetBlueprint'/Game/UI/widget/HpBar.HpBar_C'"));
    WidgetComponent->SetWidgetClass(Widget);
}

```

我们先初始化了WidgetComponent，然后把他添加到角色身上。接着我们读取了我们之前创建的蓝图控件，然后设置给了WidgetComponent(路径可以通过<font color=#dd1144 size=3>右键点击</font>控件蓝图，选择<font color=#dd1144 size=3>复制引用</font>)

>**<font color=#DC143C size=4>注意：这里有个大坑，动态加载UClass时需要在资源路径结尾增加“_C”</font>** 

编译你的代码，你的角色就可以显示出血条了：

![示例图片](http://img.blog.csdn.net/20161125114330279)

## **<font color=#191970 size=5>血条的更新</font> ** ##

血条已经显示出来来了，接下来的工作就是怎么来更新他的长度，我们先要找到控件：

- <font color=#dd1144 size=3>MyCharacter.h</font>

``` cpp
public:
    // 在初始化完组件之后调用
    virtual void PostInitializeComponents() override;

    // HpBarWidget
    UPROPERTY()
    UProgressBar* HPBarProgress;
```

我们需要在<font color=#dd1144 size=3>PostInitializeComponents</font>函数中找到控件，因此我们重写了此方法。然后定义了一个<font color=#dd1144 size=3>UProgressBar</font>指针来存储它，我们来看看实现部分：

- <font color=#dd1144 size=3>MyCharacter.cpp</font>

``` cpp
void AMyCharacter::PostInitializeComponents()
{
    Super::PostInitializeComponents();

    UUserWidget* CurrentWidget = WidgetComponent->GetUserWidgetObject();

    if ( NULL != CurrentWidget )
    {
        HPBarProgress = Cast<UProgressBar>(CurrentWidget->GetWidgetFromName(TEXT("HPBarProgress")));
        if (NULL != HPBarProgress)
        {
            HPBarProgress->SetPercent(0.1f);
        }
    }
}
```

这个函数中我们先获得了<font color=#dd1144 size=3>WidgetComponent</font>当前正在显示的控件，然后我们在控件里用<font color=#dd1144 size=3>GetWidgetFromName</font>方法找到了血条的控件。然后为了方便测试，在不为空的时候，我们把血量设置为了还有10%。现在我们成功找到了并存储了血条控件。你可以在你想要的地方更新血条的长度，只需要调用下面的方法：

``` cpp
HPBarProgress->SetPercent(HpPercent);
```

这就完成了血条部分的更新。
