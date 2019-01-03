---
title: "Willin: 大翼航空：云上自动化测试与持续集成实践"
date: 2018-01-03 14:38:30
categories: Dev
tags: [node.js, fe, be, dev]
---



# 大翼航空：云上自动化测试与持续集成实践

## 公司简介

大翼航空成立于2015年，致力于无人机及其应用管理平台的研发、销售和服务，是DJI大疆创新系统集成商、senseFly授权经销商。

公司主营自主研发的“风筝线”无人机管理系统，将无人机视作空中物联网平台。“风筝线”系统注重空中数据的精准采集、智能分析与实时呈现。在“一云一网一图”的框架和成熟飞行平台的基础上，着力于应用软件的研发，打造包含采集、分析与展示的“端到端”一体化解决方案，帮助客户进行高效的分析及决策，满足行业需求。

目前，我公司的无人机云系统和服务已经为公安、消防、环保、农业等客户带来较大的效率提升，也在 “智慧城市”中展现了极大价值。随着无人机技术的成熟，网络通信技术的发展，传感器等负载的多样化，未来将应用于更多行业。

<!--more-->

## 原始架构

“风筝线”无人机云系统，包括“无人机”和“云系统”两大部分。其中：

- “无人机”部分，包括硬件（飞机+摄像头+传感器）、固件、软件。软件作为控制中枢，（通过固件）负责：（1）从云服务器接受指令；（2）向无人机发送指令；（3）从传感器手机数据；（4）将数据回传云服务器。
- “云系统”部分，主要是服务器，包括：（1）通讯服务器，负责与无人机软件交互；（2）数据库服务器，持久化数据；（3）web服务器，运行实时监控和管理系统；（4）视频服务器，接受无人机软件的推流并传递给实时监控。

整体架构图如下：

![架构图](https://user-images.githubusercontent.com/1890238/50625434-2052ff80-0f63-11e9-94b4-e51c902f77ad.jpg)

## 特殊性及方案

看似并不复杂的总体结构，实施时还是会遇到一些困难。这主要是由于无人机这一特殊载体决定的。

- 流量集中在白天，尤其是工作时间，晚上几乎没有任何流量。
- 流量的波动非常大，波峰时有数百上千架飞机同时传送数据，波谷时可能只有数架甚至没有。
- 数据传输主要通过4G网络，网络状况可能不稳定。
- 传输的位置、状态等数据频次高、体积小。
- 部分情况下，要求拍摄视频并同步传输到实时监控。

经过认真调研，采取的方案为：

- 手机软件与通讯服务器的协议为 MQTT ，而非常规的 TCP/UDP 。MQTT 适用于在网络不稳定的情况下，多频次传输少量数据。
- 通讯服务器使用消息队列，初级阶段使用 Redis 模拟消息队列。后续随着流量的增长，将使用 Kafka 等专业消息队列软件。
- 手机软件与通讯服务器使用 socket 建立长连接，并允许中断，中断后可续传。
- 视频拍摄后，直接保存到手机中，只在网络情况良好时，才上传到视频服务器。

考虑到以上的诸多特殊性，“风筝线”使用 Node.js 作为开发语言，主要原因是：

- Node.js 特别适用于高并发的情况
- 前后端统一，便于减少开发的复杂度。

在服务器的使用上，从一开始就没有决定自建服务器，主要是自建服务器需要大量的维护成本，这对于初创公司来说并不是一件轻松的事情。
因此，通用服务器、数据库、Redis 、CDN 等均直接购买了阿里云的服务。

阿里云除了商业服务外，还提供一些免费的工具，例如：CI工具 CodePipeline 。

阿里云CodePipeline是一款提供持续集成/持续交付能力，并完全兼容Jenkins的能力和使用习惯的SAAS化产品。通过使用阿里云CodePipeline，可以方便的在云端实现从代码到应用的持续集成和交付，方便快速的对产品进行功能迭代和演进。
尤其是，CodePipeline 支持的数种语言中恰好包含 Node.js 。只要开发人员将代码上传到 git 服务器的 master 分支，就会自动触发构建和测试过程，测试通过后，自动部署到CDN。

## 迁移过程

在2017年，考虑到国际化业务的可能性需要，购买了境外某Paas服务。实际运行了大半年后发现，访问境外服务器有比较严重的延迟，对于业务有比较严重的影响。因此，在考察了国内各家 Paas 之后，我们选择了阿里云。

选择阿里云是基于如下几个方面的考虑：

- 市场评价：阿里云属于云服务商的第一梯队。
- 服务的可靠性和稳定性：阿里云拥有为淘宝等超大型网站服务的经验。
- 基础组件的完备性：如核心的RDS数据库支持，Redis以及MQ等等这些服务。阿里云拥有这一系列非常丰富的产品，其基础组件的产品线也非常完善。

### 新架构



![1-1](https://user-images.githubusercontent.com/1890238/50626176-b4bf6100-0f67-11e9-85e0-22ed70358811.jpg)

首先，我们采用了前后端完全分离的方式。

网站、系统的前端部分采用 OSS（对象存储）+ CDN（全站加速）完全静态化的托管到阿里云上。

后端服务，部分沿用原有的方式，用多台虚拟服务器 + SLB（负载均衡）进行访问控制；部分采用函数计算（Severless 架构）进行尝试。

### 自动化测试与持续集成

之前我们的自动化测试采用的是 Git 钩子方式，在 PreCommit 阶段进行，这样的话有一个问题，我们自己的开发电脑配置并不高，随着测试用例的增加，每次测试时间越来越长，严重影响了开发效率。所以我们决定将测试放到提交后执行。

![1-2](https://user-images.githubusercontent.com/1890238/50626177-b4bf6100-0f67-11e9-867d-d0bb977b8a0d.jpg)






### CodePipeline 配置

- 首先，配置源码管理，确定需要构建的代码仓库及代码分支
- 配置构建触发器，当提交 Pull Request 的时候，触发执行
- 构建脚本执行测试 `npm test`
- 如果为前端项目，还需要执行 Webpack 构建 `npm run build`
- 如果测试失败或构建失败，停止并通知
- 如果为前端项目，构建成功后上传至 OSS Bucket
- 如果为后端项目，执行远程服务器热更新脚本 `ssh -o StrictHostKeyChecking=no  [忽略参数] "NODE_ENV=production node /PATH/ci"`
- 构建结果通过钉钉通知到开发群



### 热更新

由于负载均衡器后有多台服务器，所以当其中一台停止更新时，流量会比自动引到其他的服务器上。

热更新脚本会顺序执行以下操作：

- 记录 Commit Hash，方便回滚
- 更新代码
- 更新依赖包
- PM2 热重启当前服务器下的服务进程



在构建过程中，除了 CodePipeline 会通知构建的结果以外，阿里云监控系统会针对整个服务的健康状态进行监控，发生异常时通过短信进行报警。我们可以通过准备好的回滚方案进行一键回滚。



整个持续集成的过程中几乎不再需要任何人工的干预。