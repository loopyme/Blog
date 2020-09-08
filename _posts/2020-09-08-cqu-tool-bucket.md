---
layout:     post
title: 重庆大学全家桶计划
subtitle: CQU Tool Bucket Program 
date:       2020-09-08
author:     Loopy
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - Fun
    - Cqu-tool-bucket

---

## 1. 简介

[重庆大学全家桶计划](https://github.com/topics/cqu-tool-bucket)是一项由[CQU-AI](https://github.com/CQU-AI)发起的开放性计划，旨在提供和持续维护一些在重庆大学生活中可能会使用到的第三方小工具。

该计划是开放的，任何组织和个人都可以不经审核的加入该项目，只需将仓库托管至GitHub并添加`cqu-tool-bucket`tag和合适的License即可。同时，我们也鼓励开发者加入[CQU-AI](https://github.com/CQU-AI)，参与[核心项目](https://github.com/orgs/CQU-AI/projects/1)的开发与维护。

## 2. 项目现状

目前[重庆大学全家桶计划](https://github.com/topics/cqu-tool-bucket)包括以下几个子项目：

### 2.1. 选课

针对重庆大学的选课流程和规则，我们分析了其痛点，并具体针对第一轮选课和第二轮选课分别开发了辅助工具。

第一轮选课时间为当前学期第15~16周，选下学期主修专业课程、通识与素质教育课程和大学英语拓展课等；第二轮选课时间为下学期开学1~4周，包括主修专业课程补退选、通识与素质教育课程补退选、非限制性选修选课、辅修选课、重修选课和刷新选课等。新生第一学期选课只设一轮（其余学期与高年级学生一致），时间为入学后的第3~4周。

#### 2.1.1. 第一轮选课-快抢

> 由于“利用抢课软件等非学校提供的选课系统选课”可能违反了《重庆大学普通本科学生违纪处分办法》，[cqu-kx](https://github.com/CQU-AI/cqu-kx)目前被闭源，仅供[CQU-AI](https://github.com/CQU-AI)组织内部交流使用。

第一轮选课的关键在于“快”，针对该问题，在技术上，我们采用选课页面部分加载和适当高频提交的方式来帮助用户选择到心仪的课程和老师。

#### 2.1.1. 第二轮选课-快选

> [cqu-kx](https://github.com/CQU-AI/cqu-kx) 重庆大学课程可选人数查询工具－“快选!!”
> [![pypi](https://img.shields.io/pypi/v/cqu-kx)](https://pypi.org/project/cqu-kx/)
>![download](https://pepy.tech/badge/cqu-kx)

第二轮选课的关键在于“捡”，在其他同学退课后，第一时间查询到课程的空余名额，就能“弯道超车”，捡到心仪的课程。

使用cqu-kx工具，就可以一键的快速查询各个课程的可选人数，以便在二次选课时，抢到被退回的名额。

同时CQU-AI无偿的提供网页查看的服务，具体可查看[WEB查看指南](https://github.com/CQU-AI/cqu-kx#21-%E7%BD%91%E9%A1%B5%E6%9F%A5%E7%9C%8B)。

### 2.2. 课程表导出

> [cqu-kb](https://github.com/CQU-AI/cqu-kb) 重庆大学课表日历生成工具
> [![pypi](https://img.shields.io/pypi/v/cqu-kb)](https://pypi.org/project/cqu-kb/)
> ![download](https://pepy.tech/badge/cqu-kb)

经验表明，经常从教务网上手动查询课表，并依靠记忆来上课，势必会导致频繁的旷课。

所以，我们开发了[cqu-kb](https://github.com/CQU-AI/cqu-kb),该工具会自动获取你的课表，并将其保存为ics文件，后者可被导入到主流的日历软件中，你就能从手机自带的日历软件上查询到课程，甚至设置上课提醒了。

类似的，CQU-AI无偿的提供日历订阅的服务，使得日历可被自动更新，具体可查看[日历订阅指南](https://github.com/CQU-AI/cqu-kb#11-%E8%AE%A2%E9%98%85%E4%BD%BF%E7%94%A8)。

### 2.3. 成绩查询

> [cqu-cj](https://github.com/CQU-AI/cqu-cj) 重庆大学成绩查询工具
> [![pypi](https://img.shields.io/pypi/v/cqu-cj)](https://pypi.org/project/cqu-cj/)
>![download](https://pepy.tech/badge/cqu-cj)

成绩查询，可以通过新教务网，WeCQU，老教务网等多种方式。根据我们的经验，这三种方式都有诸多缺点。

于是，我们开发了[cqu-cj](https://github.com/CQU-AI/cqu-cj)，该查询工具能：
 - 直接生成csv格式的成绩表（可以用excel打开）
 - 无论评教与否都能查询入学以来的所有成绩
 - 查询到的数据段包括`课程编码`，`课程名称`，`成绩`，`学分`，`选修`，`课程类别`（专选，必修等），`教师`，`考试类别`，`备注`，`考试时间`

### 2.4. 上网

> [DrcomExecutor/cqu-de](https://github.com/CQU-AI/DrcomExecutor) 重de庆大学 DrCOM 登录器
> [![pypi](https://img.shields.io/pypi/v/cqu-de)](https://pypi.org/project/cqu-de/)
>![download](https://pepy.tech/badge/cqu-de)

官方DrCOM登录器无法稳定开热点，而且在linux上很不友善，还丑！而且不具有自动重连等机制，极大的影响了使用体验。

于是，我们开发了[DrcomExecutor/cqu-de](https://github.com/CQU-AI/DrcomExecutor)，该工具能：
 - 稳定开热点
 - 自动查询剩余流量与付费组
 - 开包即用，直接输入用户和密码，无需配置
 - 完美支持Mac和Linux,在Windows上也能稳定运行
 - 自动的断线重连，基于指数退避的全自动重连。真正实现“只要有网就能上网”。

### 2.5. 面对开发者的子项目

目前[重庆大学全家桶计划](https://github.com/topics/cqu-tool-bucket)包括以下几个面对开发者的子项目：
- [cqu-jxgl](https://github.com/CQU-AI/cqu-jxgl): fork自[maxoyed/cqu_jwc](https://github.com/maxoyed/cqu_jwc/tree/bd09b6a433f1a50794982548c23fa014710a0a39)，经过了重构调整和优化，能完美处理重庆大学教务处相关的底层网络操作，并能自动处理DSafeId cookie机制。
- [cqu-tool-buckect-template](https://github.com/CQU-AI/cqu-tool-buckect-template)：模板仓库，存储了Git Action和config等项目配套设施。

## 3. 声明

 - 本项目完全用爱发电，暂时没有sponsor
 - 所有子项目开放源代码，可自行检查是否窃取你的信息。
 - 所有子项目都不存储用户的帐号，密码。
 - 所有子项目都不存储任何人的成绩，课表等相关信息，所有的数据来自于重庆大学相关网站。

## 4. 相关资料与链接

 - [项目清单](https://github.com/topics/cqu-tool-bucket)
 - [CQU-AI](https://github.com/orgs/CQU-AI)
 - [核心项目看板](https://github.com/orgs/CQU-AI/projects/1)
 - 使用指南可查询各项目具体的Readme
