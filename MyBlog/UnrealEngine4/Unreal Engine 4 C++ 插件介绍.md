[TOC]

# **Unreal Engine 4 C++ 插件介绍**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.创建插件</font> ** ##

创建插件的步骤很简单，但是很容易出错：

> 1. <font color=#dd1144 size=3>关闭项目，在项目目录下创建Plugins文件夹</font>
> 2. <font color=#dd1144 size=3>拷贝一个空白插件（BlankPlugin）到Plugins文件夹下（BlankPlugin位于路径Engine/Plugins/Developer/BlankPlugin）</font>
> 3. <font color=#dd1144 size=3>把文件夹，文件名，文件内容里面所有的 BlankPlugin 替换为你的插件名字</font>
> 4. <font color=#dd1144 size=3>重新生成项目</font>
> 5. <font color=#dd1144 size=3>重新编译项目</font>

会发现你的项目中已经自动检测出了插件：

![项目示例](http://img.blog.csdn.net/20161212105016594?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjAzMDk5MzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中<font color=#dd1144 size=3><插件名称>.uplugin</font>为插件的描述文件：

``` cs    
{
    "FileVersion" : 3,
    "Version" : 1,
    "VersionName" : "1.0",
    "FriendlyName" : "MyTestPlugin",
    "Description" : "Test Plugin Develop",
    "Category" : "Tests",
    "CreatedBy" : "Colocasia",
    "CreatedByURL" : "",
    "DocsURL" : "",
    "MarketplaceURL" : "",
    "SupportURL" : "",
    "EnabledByDefault" : true,
    "CanContainContent" : false,
    "IsBetaVersion" : false,
    "Installed" : false,
    "Modules" :
    [
        {
            "Name" : "MyTestPlugin",
            "Type" : "Developer",
            "LoadingPhase" : "Default"
        }
    ]
}
```

描述文件不能乱改，否则会导致加载失败，需要修改可以参考官方文档：[描述器文件格式](https://docs.unrealengine.com/latest/CHN/Programming/Plugins/index.html#%E6%8F%8F%E8%BF%B0%E5%99%A8%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F)

插件中也有项目的配置文件，<font color=#dd1144 size=3><插件名称>.Build.cs</font>：

``` cs    
// Copyright 1998-2016 Epic Games, Inc. All Rights Reserved.

namespace UnrealBuildTool.Rules
{
    public class MyTestPlugin : ModuleRules
    {
        public MyTestPlugin(TargetInfo Target)
        {
            PublicIncludePaths.AddRange(
                new string[] {
                    // "MyTestPlugin/Public"
                    // ... add public include paths required here ...
                }
            );

            PrivateIncludePaths.AddRange(
                new string[] {
                    "MyTestPlugin/Private"
                    // ... add other private include paths required here ...
                }
            );

            PublicDependencyModuleNames.AddRange(
                new string[]
                {
                    "Core",
                    "CoreUObject"
                    // ... add other public dependencies that you statically link with here ...
                }
            );

            PrivateDependencyModuleNames.AddRange(
                new string[]
                {
                    // ... add private dependencies that you statically link with here ...
                }
            );

            DynamicallyLoadedModuleNames.AddRange(
                new string[]
                {
                    // ... add any modules that your module loads dynamically here ...
                }
            );
        }
    }
}
```

然后配置完后就可以像引擎编写模块一样，愉快的编写代码了。

----------

## **<font color=#191970 size=5>2.C++静态链接插件</font> ** ##

在说此方法之前，先引用下官网的说明，不是很推荐用这种方法：

>为了使插件真正可插拔，我们通常不鼓励为插件添加依赖关系。

但是如果一个游戏各个系统都是一个个插件，或者有人发布了很好用的代码类的工具插件，就可以用下面的方式进行静态链接。（静态链接后如果找不到插件编译会报错）

在你需要依赖的模块的<font color=#dd1144 size=3><模块名称>.Build.cs</font>中加入下列代码：

``` cs    
PrivateDependencyModuleNames.AddRange(
    new string[] {
        "MyTestPlugin"
    }
);

PrivateIncludePathModuleNames.AddRange(
    new string[] {
        "MyTestPlugin"
    }
);
```

这样就可以访问插件中的接口了，不过这里有个大坑：

>**<font color=#DC143C size=4>插件中的要让其他模块访问的接口要符合模块API的标准</font>**

这个问题曾经困扰了我很久：

``` cpp   
/**
 * Example of declaring a UObject in a plugin module
 */
UCLASS()
class MYTESTPLUGIN_API UMyTestObject : public UObject
{
    GENERATED_UCLASS_BODY()
};
```

比如这个例子中：你想<font color=#dd1144 size=3>UMyTestObject</font>被游戏中的模块创建，必须加入<font color=#dd1144 size=3>MYTESTPLUGIN_API</font>这个标记，否则会报无法找到定义的LINK2019错误。加入的标记符合<font color=#dd1144 size=3><插件名称>_API</font>，注意插件名称必须为大写。
