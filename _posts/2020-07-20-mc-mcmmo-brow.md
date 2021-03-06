---
layout:     post
title: 一种适用于mcMMO的四阶段流水线多路并发炼药机设计与实现
subtitle: 带着脑子玩游戏-mc中的模拟电路与流水线
date:       2020-07-20
author:     Loopy
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - Fun
    - Minecraft

---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ['\\(','\\)'] ],
      processEscapes: true
    }
  });
  </script>
<script type="text/javascript" async src="//cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

> [LostEpoch@傻虾公益服](mailto:Peter@mail.loopy.tech)
>
> 本文介绍了在游戏我的世界中酿造的相关情况，针对搭载了mcMMO插件的特殊服务器环境，提出了一种适用于mcMMO的四阶段流水线并发炼药机设计，并在实现后进行相关的基准测试。该设计在经验获取方面存在缺陷。


## 1. 引言

从我初中开始玩SurvivalCraft以来，我花了不少娱乐时间在玩SurvivalCraft，Minecraft这一类的游戏，但一直玩得断断续续的，而且都是在手机上，直到前几段时间被朋友安利了Java版，买来玩了以后发现和以前手机上的大不相同，现在主要是在一个叫`傻虾公益服`的纯净生存服务器中玩耍。该服务器中安装使用了mcMMO插件，我在游戏过程中，突然深感共产主义的优越性以及共产主义社会的超越性和现实性，为了充分消除剥削，并对服务器的共产化做一些*微小的工作*，决定无偿给其他玩家供给药水，由此引出了一个适用于mcMMO的超高产量的AFK酿造机需求。

作为一个带着脑子玩游戏的玩家，我决定充分使用模拟电路和计算机系统结构设计的知识设计一个适用于mcMMO的四阶段流水线多路并发炼药机。

（感觉`我的世界`被我玩成了仿真器...就像在Vivado里设计实现CPU一样，我在`我的世界`从基本逻辑门开始设计实现了一个复杂高效而且鲁棒的生产机械）


## 2. 游戏环境
### 2.1. 我的世界
我的世界（Minecraft）是一款沙盒式建造游戏，缔造者为Mojang Studios创始人马库斯·佩尔松，其灵感源于《无尽矿工》､《矮人要塞》和《地下城守护者》。该游戏着重于让玩家去探索、交互、并且改变一个由一立方米大小的方块动态生成的地图。除了方块以外，环境单体还包括植物、生物与物品。游戏里的各种活动包括采集矿石、与敌对生物战斗、合成新的方块与收集各种在游戏中找到的资源的工具。游戏中的无限制模式让玩家在各种多人游戏服务器或他们的单人模式中进行创造建筑物、作品与艺术创作。其他功能包括逻辑运算与远程动作的红石电路、矿车及轨道。

### 2.2. 药水酿造

药水酿造（Brewing）是指在酿造台中在水瓶中加入各种的材料来制作药水、喷溅药水和滞留药水的过程。将1-3个药水瓶放入酿造台界面底部的3个药水槽中，将材料放入顶部的材料槽中，在燃料槽里放置烈焰粉，便可以将材料提炼到药水瓶中，并酿造出能给予玩家效果的药水。

![brewing.png](https://gamepedia.cursecdn.com/minecraft_zh_gamepedia/thumb/1/12/MinecraftPotionsClean.png/400px-MinecraftPotionsClean.png?version=b4e8ef6ccfddbe66ab862c0a7ae36d91)

如上图所述，每种药水都是从制作水瓶开始的，水瓶可以通过玻璃瓶从水源或炼药锅里装水而获得。在烈焰粉的供能下，下一步则是添加基础材料来制造基础药水，通常会是下界疣，它会酿造出粗制的药水，但本身并不会给药水添加效果。当你以同样的方式放入二级材料时，你就可以制造出有效果的药水了。放入第三个材料可以使药水效果增强、持续更久、或者是反转效果；在酿造的任意阶段加入火药，可以将药水转化成喷溅药水，使药水能被投掷（或被发射器发射）并使效果扩散开来。最后，可以对任意阶段的喷溅药水加入龙息，将药水转化成滞留药水，使药水效果可以在一个圆形范围内持续一段时间。每个酿造步骤耗时20秒。

### 2.3. mcMMO

mcMMO是《我的世界》著名的服务器端Mod，下载量超过290万次。mcMMO将技能机制引入到《我的世界》中，增强了原生的游戏机制，扩展了原生的游戏玩法。mcMMO采用了我的世界核心游戏机制，并将其扩展为一种广泛而高质量的RPG模式游戏。目前，mcMMO增加了14种独特的技能来训练和升级。

![mcmmo.png](https://mcmmo.org/w/images/0/02/McMMO_Spigot_Banner.png)

本文所述的主要是炼金术技能（Alchemy），它是一项以酿造药水为基础的杂项技能。技能的升级主要通过大量的酿造药水。这项技能使得玩家可以更迅速的酿造药水，同时还能获得其他功能的药水。炼金术技能不具有超级技能或其他主动技能，只有两个被动技能：催化(Catalysis)和调合(Concoction)。

催化(Catalysis)会显著提升原料溶入药水的速度，从而加速整个酿造进程。催化在100级时默认解锁， 1000级时达到原来的四倍。调合(Concoction)则使得药水可以使用自定义的原料来酿造， 也即添加了更多的药水配方。不同阶段会有不同炼金原料被解锁。具体等级可参照下表（等级不唯一，在不同的版本下可能不同，根据管理员的设置也可能不同）：

|药水名称|药水效果|成分|解锁等级|类型|信息|
|---|---|---|---|---|---|---|---|
|水下呼吸药水|水下呼吸|睡莲|1||允许水下呼吸|
|急迫药水|急迫|胡萝卜|250|增益|提高采矿速度|
|挖掘疲劳药水|挖掘疲劳|粘液球|250|减益|降低采矿速度|
|吸收药水|吸收伤害|石英|375|增益|每级增加2个额外的不可再生的生命值|
|跳跃药水|跳跃增强|红蘑菇|375|增益|增加跳跃高度|
|生命药水|生命|苹果|500|增益|每级增加2个额外的可再生的生命值|
|饥饿药水|饥饿|腐肉|500|减益|更快饥饿|
|恶心药水|恶心|棕色蘑菇|625|减益|使视线扭曲|
|失明药水|失明|墨囊|625|减益|降低使用者的视野|
|饱和药水|饱和|蕨|750|增益|导致用户的饥饿感增加而不消耗食物|
|腐烂药水|枯萎|毒土豆|875|减益|使使用者受伤，类似于毒药|
|抗性药水|抗性提升|金苹果|1000|增益|每级施加20％的伤害减免|

提升等级有以下四种途径：
1. 第一阶段酿造（瓶装水到粗制药水）：每瓶15xp
2. 第二阶段酿造（粗制药水到其他药水）：每瓶30xp
3. 第三阶段酿造（使用红石或荧石）：每瓶60xp
4. 第四阶段酿造（使用火药制作喷溅药水）：每瓶120xp

同时，酿造的界定是玩家的投料行为，一旦用户投料，则认为该酿造行为是玩家触发的。也就是说，使用漏斗来自动投料是无法给玩家积累经验的。

## 3. 需求情景

该酿造机需要：
1. **AFK**: 目标产量极大，而每个人只有一个肝，手动步骤过多的酿造的是无法被接受的。
1. **并发执行**: 每个酿造流水线单元都是单片的，使得各个酿造流水线单元可以被堆叠。
2. **手动投料**: 所有原料必须玩家手动投入，玩家才能积累经验，并且酿造出一些mcMMO特殊添加药水配方。

## 4. 原型设计
### 4.1. 整体设计

该系统设计基于流水线架构，单级流水线只执行单一的功能，这将使得每级流水线投料可以以64个（组）为单位，减少手动投料的频次。通过红石信号控制漏斗来在不同的流水线阶段中传递原料。由于投料是玩家批量投入的，mcMMO会认为酿造行为是玩家触发的，也就使得玩家能积累经验，并且酿造出一些mcMMO特殊的药水配方。

酿造系统由两个模块组成：信号产生模块，酿造流水线执行模块。

![wave.png](https://blog.loopy.tech/img/2020-07-20/wave.png)

信号产生模块需要产生上图中Stage1,Stage2,Stage3,Stage4所需的信号波形（也即是各级流水线所需的控制信号）。经过信号分解，发现可以通过两个带相位差的方波信号（CLK1与CLK2）进行同或运算得到（CLK_XNOR）基准信号，再经过多级延迟产生所需的所有波形.
 - $$S_{CLK2} = S_{CLK1} +t$$
 - $$S_{Stage1} = S_{CLK1} \bigodot S_{CLK2}+0 \times t$$
 - $$S_{Stage2} = S_{CLK1} \bigodot S_{CLK2}+1 \times t$$
 - $$S_{Stage3} = S_{CLK1} \bigodot S_{CLK2}+2 \times t$$
 - $$S_{Stage4} = S_{CLK1} \bigodot S_{CLK2}+3 \times t$$

酿造流水线执行模块接收信号产生模块传递的四个信号，按照上图标注的工作模式运行。

### 4.2. 理论产量

由于流水线可并发堆叠，理论产量是无穷大的，本节将只对4路并发的酿造系统进行分析，同时将对每一生产批次（玩家投料后可以AFK的时间周期）进行分析。

在每一生产批次$T$中，产量主要受限于投料堆叠数，即 $\sigma=64$ ,每一路酿造流水线单元只能制造$3\sigma$瓶药水。对单个生产批次的产量$Count$和所需的时间$T$则有

$$ Count = 3 \times \sigma $$

$$T=\sigma \times (t_{单次酿造时间}+t_{单瓶药水收集时间} \times 3 + t_{收集屏蔽时间}) + t_{信号起振延迟}$$

其中$t_{单次酿造时间}$随玩家炼金等级提高而降低，在本节计算中按照最大时间$20s$进行计算。$t_{收集屏蔽时间}$设计为$0.4s$。

单生产批次下，生产速度为$\frac{Count}{T} \approx 0.55$瓶每秒，也就是每一生产批次23分钟，产出768瓶（25.6箱）药水。

### 4.3. 原型测试

首先在创造模式的超平坦世界中进行了搭建与原型测试，以下为测试截图。

1. 信号产生模块
    ![clk.png](https://blog.loopy.tech/img/2020-07-20/clk.png)

2. 酿造流水线执行模块
    ![pipeline.png](https://blog.loopy.tech/img/2020-07-20/pipeline.png)
    ![pipeline1.png](https://blog.loopy.tech/img/2020-07-20/pipeline1.png)

3. 水瓶灌装系统（参考）
    ![water.png](https://blog.loopy.tech/img/2020-07-20/water.png)

4. 运行

  酿造时整体运行截图

  ![brewing.png](https://blog.loopy.tech/img/2020-07-20/brewing.png)
  收集信号触发时整体运行截图
  ![collecting.png](https://blog.loopy.tech/img/2020-07-20/collecting.png)

在原型测试中，整个酿造系统能够按照设计进行运作，实际产出与计算的设计理论产出一致。

## 5. 实现测试

~~修好了还没测试，服搬家了...~~

1. 第一层：收集及存储层
    ![test-1.png](https://blog.loopy.tech/img/2020-07-20/test-1.png)
1. 第二层：信号产生层
    ![test-2.png](https://blog.loopy.tech/img/2020-07-20/test-2.png)
1. 第三层：酿造执行层
    ![test-3.1.png](https://blog.loopy.tech/img/2020-07-20/test-3.1.png)
    ![test-3.2.png](https://blog.loopy.tech/img/2020-07-20/test-3.2.png)
1. 第四层：下界疣种植和水瓶存储层
    ![test-4.png](https://blog.loopy.tech/img/2020-07-20/test-4.png)
1. 第五层：水瓶手动灌装层
    ![test-5.png](https://blog.loopy.tech/img/2020-07-20/test-5.png)

在实现测试中，整个酿造系统基本能够按照设计进行运作，实际药水产出与计算的设计理论产出一致。但经验获取存在问题，mcmmo计数时无法计数批量投料，需要结合Python脚本进行投料欺骗，即每周期取出并重新放入原料。

## 6. 致谢
本文受到了`LostAdder@傻虾公益服`和`Da_yang12138@傻虾公益服`的帮助,接受了`DunDun村民交易所@傻虾公益服`的资助和支持。

## 7. 参考
1. 我的世界官方网站：https://www.minecraft.net
1. GamePedia-我的世界：https://minecraft-zh.gamepedia.com/
2. GamePedia-药水酿造：https://minecraft-zh.gamepedia.com/%E8%8D%AF%E6%B0%B4%E9%85%BF%E9%80%A0
3. mcMMO官方网站：https://mcmmo.org/wiki
4. mcMMO炼金术：https://mcmmo.fandom.com/wiki/Alchemy

## 8. 附录

### 8.1. 生产方式

生电机械的宗旨就是充分使用生产资料来创造利润。玩家在进行资本的原始积累后，迅速将其转化为生产资料，以加快资本的积累，其根本原理是通过极高的机械生产力以替代玩家劳动力，避免“肝”的流失。

生电玩家的游戏过程以使用机器的大生产为特征，通过快速的发展和科技迭代以积累难以想象巨额的资本，并通常伴随着对其他玩家“肝”资源的剥削和劳动力的强制获取。

而资本主义生产方式的出现，是空想社会主义产生的现实前提。社会主义,无论是哪一种的社会主义，都认为应当由社会控制生产资料。不带市场的社会主义追求避免资本积累，而市场社会主义强调在社会拥有的企业和资本分配中保留金钱、要素市场甚至利润动机的作用。这些企业的利润可以选择直接由雇员控制，或者选择以社会红利的形式发给全社会。

在Minecraft服务器中，想要建成社会主义社会，就需要消除个人所有制，按需获取资源，这在理论上是可以被实现的（以我所知晓的机械和资本，足以供给全服玩家进行正常的生产生活），但实际上是无法实现的。一方面，原来的被剥削阶级中有一定数量的“熊孩子”玩家，它们可能不具有操作生电机械所必需的知识储备，也可能出于独特的心理因素，而对生产资料和相关机械造成难以挽回的破坏和损失;另一方面，原来的资产阶级中有一定数量的对消除个人所有制有极大抵触的玩家，主观上不作出任何的利它行为，甚至对“一个箱子”，“一组木板”等极廉价的物品极为珍惜，并借以进行道德谴责，这类玩家是自然的，这种把自己的考虑或利益置于别人的利益之上的心理利己主义被伊壁鸠鲁学派认为是人类与生据来的。

### 8.2. 科技树

以下记录生电机械的修建科技树（已全部在`傻虾公益服`中实现，由于该服务器对刷怪存在限制，科技树中基本不考虑刷怪型机械）

<script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/8.0.0/mermaid.min.js"></script>
<script>mermaid.initialize({startOnLoad:true});</script>
<div class="mermaid">
graph TD

    出生>出生]

    全自动机械(全自动机械)
    半自动机械{半自动机械}
    事件[事件]
    状态>状态]

    资本的原始积累[资本的原始积累]
    杀凋[杀凋]
    下界刷怪[下界刷怪]
    卖南瓜[卖南瓜]
    mcmmo拆解[mcmmo拆解]
    
    甘蔗机(甘蔗机)
    分捡机(分捡机)
    繁殖和烤鸡机(繁殖和烤鸡机)
    繁殖和烤牛机{繁殖和烤牛机}
    全颜色羊毛机(全颜色羊毛机)
    并发药水酿造机((并发药水酿造机*))
    南瓜机(南瓜机)
    全种类村民交易所{全种类村民交易所}
    工业熔炉(工业熔炉)
    急迫刷石机{急迫刷石机}
    刷铁机(刷铁机)
    村民繁殖机(村民繁殖机)
    刷花机{刷花机}
    刷海泡菜机{刷海泡菜机}
    潜影盒灌装机(潜影盒灌装机)
    村民田(村民田)
    仙人掌塔(仙人掌塔)

    大量物品进入创造模式>大量物品进入创造模式]
    药水进入创造模式>药水进入创造模式]
    无限绿宝石+经验>无限绿宝石+经验]
    全附魔书村民收集[全附魔书村民收集]
    满附魔装备变成廉价绿宝石产品>满附魔装备变成廉价绿宝石产品]
    钻石等物品可被制造>钻石等物品可被制造]

    出生-->资本的原始积累
    资本的原始积累-->甘蔗机
    资本的原始积累-->刷花机
    资本的原始积累-->仙人掌塔
    资本的原始积累-->刷海泡菜机
    资本的原始积累-->获得村民
    资本的原始积累-->繁殖和烤鸡机
    资本的原始积累-->工业熔炉
    资本的原始积累-->繁殖和烤牛机
    村民繁殖机-->村民田
    村民繁殖机-->刷铁机
    资本的原始积累-->杀凋
    资本的原始积累-->下界刷怪
    资本的原始积累-->分捡机
    获得村民-->村民繁殖机
    杀凋-->急迫刷石机
    分捡机-->全颜色羊毛机
    刷铁机-->全颜色羊毛机
    急迫刷石机-->南瓜机
    南瓜机-->卖南瓜
    卖南瓜-->无限绿宝石+经验
    无限绿宝石+经验-->全种类村民交易所
    村民繁殖机-->全种类村民交易所
    甘蔗机---全种类村民交易所
    村民田-->并发药水酿造机
    全颜色羊毛机---全种类村民交易所
    大量物品进入创造模式-->并发药水酿造机
    全种类村民交易所-->大量物品进入创造模式
    全种类村民交易所-->全附魔书村民收集
    全附魔书村民收集-->满附魔装备变成廉价绿宝石产品
    满附魔装备变成廉价绿宝石产品-->mcmmo拆解
    全种类村民交易所-->mcmmo拆解
    mcmmo拆解-->钻石等物品可被制造
    工业熔炉-->并发药水酿造机
    下界刷怪-->并发药水酿造机
    并发药水酿造机-->药水进入创造模式
    药水进入创造模式-->潜影盒灌装机
</div>
