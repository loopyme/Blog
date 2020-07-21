---
layout:     post
title: 深度学习模型计算图相同子结构的识别和展示
subtitle: Mindspore-Subgraph Detection
date:       2020-07-10
author:     Loopy
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - Algorithm

---

最近一小段时间没有好玩的代码可以敲了，于是抽空参加了一个[“开源软件供应链点亮计划-暑期2020”](https://isrc.iscas.ac.cn/summer2020/#/index)，给华为的MindSpore设计一个在计算图中寻找相同子结构的算法并且写一个demo出来。

## show me the code

Repo主要托管在Github上，但因为计划要求需要存到软件所的一个Gitlab上，而MindSpore是托管在了Gitee上了，所以镜像挺多的，用github action自动镜像， ![Mirror](https://github.com/loopyme/mindspore_subgraph_detection/workflows/Mirror/badge.svg)：
 - Gitee： https://gitee.com/loopyme/mindspore_subgraph_detection
 - GitHub：https://github.com/loopyme/mindspore_subgraph_detection
 - GItlab：https://isrc.iscas.ac.cn/gitlab/summer2020/students/proj-2017182

现在最新的Release在![GitHub release (latest by date)](https://img.shields.io/github/v/release/loopyme/mindspore_subgraph_detection)

文档的话，用git page怕访问贼慢（甚至可能被墙），所以丢到了Gitee page上：https://loopyme.gitee.io/mindspore_subgraph_detection/

> BTW， 该项目受到 [“开源软件供应链点亮计划-暑期2020” ](https://isrc.iscas.ac.cn/summer2020/#/index) 的资助和支持,由peter@mail.loopy.tech设计开发，由gaohan19@huawei.com指导。

## 我在干啥

MindSpore是华为自研的深度学习框架，其中的计算图模式是一种业界主流的用来进行数据传递与计算的形式。计算图主要包含节点和有向边，节点表示计算和控制操作，边表示数据的流向和控制等关系。计算图的高效合理展示，有助于用户更好的理解模型结构、发现和调试模型训练过程中出现的问题。然而，大型深度学习模型往往有着复杂的计算图结构，包含有成千上万个节点和更多的边。在这些点和边之中，包含有许多结构相同或高度相似的子结构，这些子结构不仅从图的拓扑结构上，甚至从深度学习语义上具有高度的相似性。快速识别大型计算图中上述的相同子结构，能够支持后续用收折、重叠等方式大幅减少页面中同时呈现的节点和边的数目，从而大幅改善计算图的展示效果。

## 为啥需要新算法

想要将频繁子图挖掘的算法思想应用于计算图中，需要对算法进行一些改进和调整。与大部分现有算法的应用场景相比，差异主要体现在:

 - 度量：大部分现有算法都是基于支持度的，即以子图在输入图中出现的次数来作为度量，而在当前应用情景下，支持度变得不那么重要，而应该以压缩输入图的程度来度量，即MDL(Minimum description length),MDL越大，算法效果越好，也即计算图的展示效果提升越显著。
 - 单图：大部分现有算法都是transaction型的，所处理的输入数据是由许多图构成的，每个图可能只包含几十到几百个顶点；而适用与计算图挖掘的large graph型算法所处理的输入数据有且只有一个大图，这个大图通常包含成千上万个顶点。
 - 无环：在一般情况下，计算图是无环的，这一特性使得计算图中的子图挖掘可能存在一些更快捷的算法。
 - 边相同：在一般情况下，计算图的所有边都是相同的，这一特性使得计算图在子图挖掘时可能可以不保存边权等边的相关信息。
 - 并行：现有的频繁子图挖掘算法大多未考虑并行运算，为了提高性能，并尽可能降低通信开销，需要对算法进行调整和创新，使得在并行场景下能快速稳定执行。

据上所述，由于计算图中的子图挖掘与大部分现有算法的应用场景具有巨大的差异，为了优化算法的效果，同时减少空间和时间占用，我们基于Apriori思想改进了现有的频繁子图挖掘算法，并使用Python进行了编码实现。

## 是怎么找的子图

可参阅[文档-算法简述](https://loopyme.gitee.io/mindspore_subgraph_detection/pages/algorithm.html)

## 现在能干啥

现在项目可以命令行，也可以Python中运行，把MIndspore的pb格式（对，就是google家的那个`protobuf`）的计算图文件进行解析，找到其中的频繁子图，然后输出到一个json文件里。在几个测试的计算图上的效果都还不错。


## 接下来干啥
主要是为了加速，同时多测试一些用例，修复一下遇到的bug：
 - [ ] 利用命名空间节点中的信息，加快第一Epoch的子图核生成
 - [ ] 尝试让子图核向上游增长