
[TOC]

# **Unreal Engine 4 C++ UCLASS构造函数易出错分析**#

好记性不如烂笔头啊，还是记录一下!

----------

在Unreal Engine 4的任意类中通常会见到两个宏：

- <font color=#BD63C5 size=3>GENERATED_BODY()</font>
- <font color=#BD63C5 size=3>GENERATED_UCLASS_BODY()</font>

在一篇教程中有时候会有这样的说法：

<a href="http://www.v5xy.com/wp-content/uploads/2015/12/11.png"><img alt="11" class="alignnone size-medium wp-image-480" src="http://www.v5xy.com/wp-content/uploads/2015/12/11-300x103.png" height="103" width="300"></a>

这个说法并不严谨，并没有完全解释UCLASS_BODY()和BODY()区别。

**<font face="微软雅黑" color=#DC143C size=3>具体分析：</font>**

首先为什么有两个宏定义的区别，主要是考虑到，继承父类之后，在于是否需要对父类的东西有所改动，构造函数的初始化亦是如此。

下面来说明下两种方法的不同：

----------

## **<font color=#191970 size=5>1.GENERATED_BODY()</font> ** ##

如果定义的是<font color=#BD63C5 size=3>GENERATED_BODY()</font>，那么意味着我不需要使用父类的构造函数，也就是说，我不能直接使用父类的声明，但是，我需要去实现的时候，我就必须自己去声明，否则就会报错。

- MyCharacter.h

``` cpp
UCLASS()
class MYCHARACTER_API AMyCharacter : public ACharacter
{
    GENERATED_BODY()
    
public:
    AMyCharacter(const FObjectInitializer& ObjectInitializer);

};
```

然后就可以在CPP中实现自己声明的这个构造函数，编译通过。

- MyCharacter.cpp

``` cpp
AMyCharacter::AMyCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{

}
```

**<font face="微软雅黑" color=#DC143C size=3>如果不去声明自己的构造函数，就会报错：</font>**

>MyCharacter.cpp(7): error C2084: 函数“AMyCharacter::AMyCharacter(const FObjectInitializer &)”已有主体
MyCharacter.h(14): note: 参见“{ctor}”的前一个定义

----------

## **<font color=#191970 size=5>1.GENERATED_UCLASS_BODY()</font> ** ##

如果定义的是<font color=#BD63C5 size=3>GENERATED_UCLASS_BODY()</font>，那么意味着我使用父类的构造函数，也就是说，我不需要为自己声明构造函数，直接去实现父类声明那个构造函数。

- MyCharacter.h

``` cpp
UCLASS()
class MYCHARACTER_API AMyCharacter : public ACharacter
{
    GENERATED_UCLASS_BODY()
};
```

那么在CPP文件中去实现，而不需要在H里面去声明，编译通过！

- MyCharacter.cpp

``` cpp
AMyCharacter::AMyCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{

}
```

**<font face="微软雅黑" color=#DC143C size=3>如果还去声明自己的构造函数，就会报错：</font>**

``` cpp
UCLASS()
class MYCHARACTER_API AMyCharacter : public ACharacter
{
    GENERATED_UCLASS_BODY()

public:
    AMyCharacter(const FObjectInitializer& ObjectInitializer);

};
```

也就是这个最常见的错误。

>error C2535: “AMyCharacter::AMyCharacter(const FObjectInitializer &)”: 已经定义或声明成员函数
note: 参见“AMyCharacter::AMyCharacter”的声明

完结..

----------

<p class="zhuanzai">本文参考：<a href="http://www.v5xy.com">少狼 – 敬畏知识的顽皮狗</a> » <a href="http://www.v5xy.com/?p=477">UE4 C++ UCLASS构造函数易出错分析</a></p>





