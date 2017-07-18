[TOC]

# **Unreal Engine 4 C++ Component介绍——BrainComponent**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>BrainComponent简介</font> ** ##

<font color=#dd1144 size=3>BrainComponent</font>是Unreal Engine 4中AI模块中的一个比较重要的组件，游戏中AI的消息收发都可以用到了这个组件。

## **<font color=#191970 size=5>添加AIModule依赖</font> ** ##

``` cs
PublicDependencyModuleNames.AddRange(
    new string[] {
        "Core",
        "CoreUObject",
        "Engine",
        "InputCore",
        "AIModule"
    }
);
```

## **<font color=#191970 size=5>继承BrainComponent</font> ** ##

- <font color=#dd1144 size=3>MyBrainComponent.h</font>

``` cpp
#pragma once

#include "BrainComponent.h"
#include "UMyBrainComponent.generated.h"
/**
 * 
 */
UCLASS()
class TEST UMyBrainComponent : public UBrainComponent
{
    GENERATED_BODY()


};
```

由于<font color=#dd1144 size=3>BrainComponent</font>是一个抽象类，所以使用它需要重新创建个类，来继承他。

- <font color=#dd1144 size=3>MyBrainComponent.cpp</font>

``` cpp

#include "UMyBrainComponent.h"

// Set Default Attribute
UMyBrainComponent::UMyBrainComponent(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{

}

```

## **<font color=#191970 size=5>使用BrainComponent</font> ** ##

然后在需要用到消息收发的<font color=#dd1144 size=3>AI控制器(AIController)</font>中初始化它。

``` cpp
BrainComponent = CreateDefaultSubobject<UMyBrainComponent>(TEXT("UMyBrainComponent"));
```

初始化后我们就可以使用它来收发消息了

## **<font color=#191970 size=5>定义消息</font> ** ##

我们可以再创建个头文件来定义消息：

``` cpp
#pragma once

#include "BrainComponent.h"

enum class MyAIState : uint8
{
    None,
    Sleep,
    Work
};

struct FMyAIMsg
{
    GENERATED_USTRUCT_BODY()

public:

    static const FName MyAIStateEnter;
};

```

## **<font color=#191970 size=5>监听消息</font> ** ##

``` cpp
// 需要监听的h中
#include "BrainComponent.h"

// 类中添加
{
    // AIMessage
    TArray<FAIMessageObserverHandle> AIMessageHandlers;    

    // Test Observer
    void TestAIMessageObserver(UBrainComponent* BrainComp);

    // Handle AI Message
    virtual void HandleAIMessage(UBrainComponent*, const FAIMessage&);
}
```

``` cpp
// 需要监听的cpp中
void MyTestBrainComponent::TestAIMessageObserver(UBrainComponent* InBrainComp)
{
    
    FName MessageType = FMyAIMsg::MyAIStateEnter;

    FOnAIMessage Delegate = FOnAIMessage::CreateUObject(this, &MyTestBrainComponent::HandleAIMessage);
    FAIMessageObserverHandle MessageHandler = FAIMessageObserver::Create(InBrainComp, MessageType, FAIRequestID::AnyRequest.GetID(), Delegate);
    

    AIMessageHandlers.Add(MessageHandler);
}

void MyTestBrainComponent::HandleAIMessage(UBrainComponent* InBrainComp, const FAIMessage& Msg)
{

    FName MsgType = Msg.MessageName;
    
    if (FMyAIMsg::MyAIStateEnter == MsgType)
    {
        if (GEngine)
        {
            // 显示调试信息五秒。-1“键”值（首个参数）说明我们无需更新或刷新此消息。
            GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Red, TEXT("HandleAIMessage"));
        }

        uint8 MsgID = Msg.RequestID.GetID();

        switch (MsgID)
        {
        case MyAIState::Sleep:
            // handle
            break;
        case MyAIState::Work:
            // handle
            break;
        default:
            // handle
            break;
        }
    }
}
```

这样就完成了消息的监听，下一步我们要发送消息。

## **<font color=#191970 size=5>发送消息</font> ** ##

发送消息就很简单了：

``` cpp
// 需要发送消息的地方

// 组装消息
FAIMessage Msg(FMyAIMsg::MyAIStateEnter, this);
Msg.RequestID = static_cast<uint32>(MyAIState::Work);
FAIMessage::Send(this, Msg);

```

之后就可以在处理函数HandleAIMessage收到消息并处理消息。（原理大家可以参考BrainComponent的源码。）