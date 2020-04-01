---
title: '[odoo]项目管理'
date: 2020-03-12 14:38:00
tags: 
- odoo
- biz
category:
- odoo
---

# 安装

通过Odoo Apps，我们可以找到project应用，直接安装，可以发现左上角菜单列表会多出Discuss和Project两项菜单。

通过查看project module详情，会发现project依赖还是比较多的，包括mail（依赖discuss modlue，所以会出现相关菜单）、portal、resouce（这个应该就是我们之前提到过的res_**系列）、web等等。

# Project使用

Project模块主要包括项目信息维护、任务信息维护。

- 任务信息维护主要采用了看板的方式，可以看到各个任务处于哪个阶段
- 我们可以通过修改Project配置启用子任务
- Project依赖analytic，提供了任务的BI分析

Odoo 12由于上线时间较久，包含了较多的project方面相关的插件；但是由于我们准备使用odoo 13，可供使用的插件较少，另外根据甲方的历史要求，这里可尝试自主开发project roles，这种对项目的管控应该还是比较需要的。

# Discuss使用

Discuss是及时通讯系统，跟国内的主流形式不大一样，不过多介绍
