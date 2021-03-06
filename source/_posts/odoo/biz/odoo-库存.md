---
title: '[odoo]库存'
date: 2020-03-17 14:31:00
tags: 
- odoo
- biz
category:
- odoo
---

# 主数据（Master Data）

# 配置

依据odoo提供的操作手册，我们可以开启很多额外的功能，这里我们增加了产品变量、产品度量、产品包装等功能，还增加了针对仓库的库位、多仓库、多路物流等。

## 多仓库
odoo默认是单仓库，可以通过配置开启多仓库，直接由界面操作即可，很方便。

## 多货位
odoo将库位类型分为了多种，这里我们可以根据这些类型，考虑odoo在设计该模块时的思想。官方文档给出的大类有三种，我感觉没有什么太大作用；另外货位类型有实有虚，这个在odoo分类中没有明显提出，而是通过构造数据来实现的，总体上看货位十分灵活。

系统中确切的分类包括以下几种, 我根据自己的理解加入的一些注释：

|货位类型|说明|
|--|--|
|Internal Location|公司内存储位置，公司进行管理|
|Vendor Location|供应商存储位置，隶属于供应商，我觉得可能使用的情况就是，供应商已经准备好了多少物资，使用方式我们在后面再讨论|
|Customer Location|客户存储位置，考虑可能的使用情况是需要记录已经发往客户的产品情况|
|View|视图存储位置，官方的示例是仓库（WH）|
|Inventory Loss|库存丢失，找不到的物资就可以放这了|
|Production|暂时不理解，说明文档里没有给出相关介绍，我觉得有两种理解方式，一是已经正在采购的物料，另一种可能是生产中正在使用|
|Transit Location|调拨中转位置，一般用于调拨（移库）操作|

## 货位组织结构
安装odoo仓储模块时会创建默认的货位数据，可以看到，这里面货位构成树形结构，通过顶层undefined中定义的Physical Locations、Partner Locations、Virtual Locations，对货位进一步分类，而树形结构中的底层货位可能会被配置到不同的公司，这种设计结构可以作为参考。
![默认货位](../../../images/1.png)

## 产品

产品（product）作为通用数据，不仅仅存在于库存模块，采购等其他模块也包含。

如果介意关心产品的存储，那么应将该产品配置为可库存；如果只是需要存储，而不关心数量变化，那么可将产品置为可消耗。

## 产品变量

## 补货规则

支持的补货规则包括两种，一是按仓位临界值补货（包括从采购补货、自己生产），一是按订单生产。

### 按仓位临界值补货
在已经配置好供应商的情况下，如果当库存数量达不到配置的最小数量：
1. 若配置产品为可采购，会生成相应采购单
2. 若配置产品为可生产，会生成相应的生产单
3. 若该产品既可采购又可生产，目前测试会从生产进行补货，需要研究是否可以部分采购、部分生产。

### 按订单生产
我的理解是订单驱动，不会有存货，订单来了才去考虑补充货物。应该和sales密切相关，还没有具体操作过。

## 包装
从包装的使用上来讲，包装的作用主要是用来对产品进行打包的，而不对应到包装发货的独立功能上，所以没有包装外再包装的打包操作，这点可以在后续的工作中进行优化

## 路径、规则与操作类型
这里的三个概念很好的描述、解决了仓储业务中的问题，是较高层次的模型抽象，在此抽象上可以建立更为自由、可扩展的功能。路径描述了完整的业务，规则描述了完整业务的切片步骤，操作类型将重复的业务切片归为同类。

### 路径
一条路径代表着一个物料流动的完整业务的可能。例如同样是描述物资从某仓库出库到客户，一条路径是物资直接从仓库出库就到达客户手中，而不经过其他的流程；另外一条路径是经过选择、打包、运输三个步骤完成物资出库。这其中每个步骤最后都会形成一个输入输出点，在该点位上可以进行对物资的入操作或出操作（这根据实际情况设定）。

### 规则
规则是依据路径而产生的“有向线段”，比如在某一个路径中货物可以由A到B，而不能由B到A。路径实际上就是由多个“同向线段”连接而成的（我偏向于将不同向的“线段”拆分成独立的路径，代表独立的业务）

### 操作类型
操作类型按照上述的比喻则更像是“有向线段”的分类，如果多条路径存在相同的“有向线段”，那么可以将这多个重复的“有向线段”归为一个操作类型，例如入库、出库