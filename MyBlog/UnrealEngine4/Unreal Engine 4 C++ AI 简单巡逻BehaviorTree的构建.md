[TOC]

# **Unreal Engine 4 C++ AI 简单巡逻BehaviorTree的构建**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.行为树(BehaviorTree)介绍</font> ** ##

我们来了解下一个完整行为树(BehaviourTree)是什么样子：

![BehaviourTree](http://img.blog.csdn.net/20151123141751776)

这个行为树(BehaviourTree)都干了些什么？他做了这些事情：

1. 平时按照固定路线巡逻
2. 发现玩家之后，追踪玩家
3. 拾取玩家视野之后，继续追踪最后一次看见的位置几秒，之后放弃
4. 放弃之后，继续按照原路线巡逻
5. 如果听到声音，警惕地向声音发生处张望几秒
6. 张望之后如果没有发现玩家，继续巡逻

***
想要看懂AI的BehaviourTree，首先要知道几个node的概念：

1. 灰色的node名为composite，是控制树走向的node
    - Selector从左到右遍历他的子树，直到一个树返回True时停止
    - Sequence从左到右遍历他的子树，直到一个树返回False时停止
2. 蓝色的node是decorator，必须内嵌在composite之中，是控制子树是否执行的判断代码（通过返回True和False）<br>decorator比较复杂，这里我们只关心他之中的ObserverAbort字段ObserverAbort可以是None，Self，LowPriority和Both
    - None时，如果decorator判定通过，就执行子树 <br>
    - Self时，如果decorator判定通过，就先停止所有正在执行的Task再执行子树 <br>
    - LowPriority时，如果decorator判定通过，就执行子树，并且不执行其右边的所有子树 <br>
    - Both就是Self和LowPriority的结合了*
3. 紫色的node是task，是AI的行为，是Blueprint代码或者C++代码，比如这里的MoveTo就是内建在BehaviourTree里的Task，而Find a waypoint就是我们在C++里边写的自定义Task，完成一些更加复杂和特殊化的任务</p>

----------

## **<font color=#191970 size=5>1.行为树(BehaviorTree)详解</font> ** ##

![BehaviourTree](http://img.blog.csdn.net/20151123141751776)
我们再次看看这个BehaviourTree，从Root结点开始，首先进入一个Selector，根据Selector的特性他会从最左边的子树执行

***

![BehaviourTree](http://img.blog.csdn.net/20151123150447178)
<br>

1. Selector下边第一棵子树，先通过访问Blackboard判断是否看到了target（玩家），如果没有，这个树返回False，根据Selector的性质，进入第二个子树
2. 如返在Blackboard中发现target字段不为null，则说明发现了玩家，进入子树判断是否知道玩家位置（其实一定是知道的。。。），最后进入MoveTo的Task，通过与Navigation寻路系统共同工作，AI将向玩家移动

***
![BehaviourTree](http://img.blog.csdn.net/20151123150457132)
<br>

1. 第二子树通过访问Blackboard判断是否听到了声音，如果没有，返回False，继续第三子树
2. 如果返回True，说明在Blackboard中ishear字段为True，则执行子树，子树是一个Sequence，所以AI会先看相声音的方向，之后wait几秒钟

***
![BehaviourTree](http://img.blog.csdn.net/20151123150512998)
<br>

1. AI大部分时间都是在执行第三子树（巡逻），第三子树通过访问Blackboard判断这个AI是不是巡逻兵，如果是，就进入子树的巡逻逻辑，如不是，返回False，AI不动
2. 如果AI是巡逻兵，进入子树，子树是一个Sequence，于是AI先发呆wait几秒（比较真实），之后执行我们自建的Find a waypoint代码，在Find a waypoint中，我们会找到一个waypoint对象，并把他赋值到Blackboard中，找到waypoint之后，AI判断是否有一个waypoint（是一定会有的，这里是为了规范再判断一次），有的话，走向waypoint，达到巡逻的效果。

***

**<font face="微软雅黑" color=#DC143C size=3>这里有几点不易理解的是：</font>**
<br>

1. BehaviourTree是每帧都执行的，而类似MoveTo一类的Task执行都是需要时间的，所以在Task执行的十几秒中，每次执行BehaviourTree，最后都会走到一样的Task之中。每次执行Task并不能直接完成任务，每次都是完成了Task中的一小点，通过每次的积累，最后完全完成Task。
2. 在执行一个Task中，发生了突发情况（比如巡逻时发现了玩家），我们会把target设置为非null的值，这是Tree会走到MoveTo那个分支，但是AI并不会直接就去追玩家，因为上一帧的巡逻Task还没跑完，所以一定会先跑完之前的Task再跑这次的Task。
3. 如果突发事件一定要立即执行，可以在decorator里加上Self的字段，这样一旦进入了这个decorator，之前的Task就全部作废，立即执行新Task。
4. 图中的第一第二子树都是Both，一是为了立即中断巡逻进行反应，而是为了不断刷新MoveTo的位置，达到紧紧追踪玩家的效果。


