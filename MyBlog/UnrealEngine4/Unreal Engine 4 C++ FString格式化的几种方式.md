
[TOC]

# **Unreal Engine 4 C++ FString操作的几种方式**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.Printf 格式化</font> ** ##
这个和C、C++的printf用法一样，先来看一看声明

``` cpp
static FString Printf
(
    const TCHAR * Fmt,
    ...
)
```
官方有详细介绍，用法如下：
``` cpp
int32 i = 1;
FString Text1 = FString::Printf(TEXT("Text%d"), i);
```
代码运行后会Text1的值为：
``` cpp
"Text1"
```

----------

## **<font color=#191970 size=5>2.Format TArray 格式化</font> ** ##
此方法用的函数为FString::Format的TArray版本，声明如下：
``` cpp
static FString Format
(
    const TCHAR * InFormatString,
    const TArray < FStringFormatArg > & InOrderedArguments
)
```
说明下参数的作用：

 - InFormatString 是进行格式化的字符串
 - InOrderedArguments 是会对InFormatString中的{index}，根据InOrderedArguments中的索引进行替换

有点难懂，直接看代码和效果会容易理解一些：

``` cpp
int32 i = 1;
TArray<FStringFormatArg> FormatArray;
FormatArray.Add(FStringFormatArg(i));
FString Text1 = FString::Format(TEXT("Text{0}"), FormatArray);
```
代码运行后会Text1的值为：
``` cpp
"Text1"
```
<br>
TArray版本还可以格式化更多的值，会用数组的第0个元素替换{0},用数组的第1个元素替换{1}。
``` cpp
TArray<FStringFormatArg> FormatArray;
FormatArray.Add(FStringFormatArg(1));
FormatArray.Add(FStringFormatArg(2));
FString Text1 = FString::Format(TEXT("Text{0}{1}{0}"), FormatArray);
```
代码运行后会Text1的值为：
``` cpp
"Text121"
```

----------

## **<font color=#191970 size=5>3.Format TMap 格式化</font> ** ##
此方法用的函数为FString::Format的TMap版本，声明如下：
``` cpp
static FString Format
(
    const TCHAR * InFormatString,
    const TMap < FString , FStringFormatArg > & InNamedArguments
)
```
说明下参数的作用：

 - InFormatString 是进行格式化的字符串
 - InNamedArguments 是会对InFormatString中的{key}，根据InNamedArguments中的键值索引替换成InNamedArguments里面的值
 
有点难懂，直接看代码和效果会容易理解一些：

``` cpp
TMap<FString, FStringFormatArg> FormatMap;
FormatMap.Add(TEXT("key1"), FStringFormatArg(1));
FString Text1 = FString::Format(TEXT("Text{key1}"), FormatMap);
```
代码运行后会Text1的值为：
``` cpp
"Text1"
```

<br>
TMap版本也还可以格式化更多的值，会用Map的键值为key1的值替换{key1},用键值为key2的值替换{key2}。
``` cpp
TMap<FString, FStringFormatArg> FormatMap;
FormatMap.Add(TEXT("key1"), FStringFormatArg(1));
FormatMap.Add(TEXT("key2"), FStringFormatArg(2));
FString Text1 = FString::Format(TEXT("Text{key1}{key2}{key1}"), FormatMap);
```
代码运行后会Text1的值为：
``` cpp
"Text121"
```

----------

## **<font color=#191970 size=5>4.字符串拼接</font> ** ##
Unreal Engine 4 的FString重载了很多操作符，可以很方便的拼接FString：

```
FString Text1 = TEXT("Text") + FString::FormatAsNumber(1);
```
代码运行后会Text1的值为：
``` cpp
"Text1"
```


