---
title: '[odoo]开发环境配置'
date: 2020-02-25 10:30:16
tags: 
- odoo
category:
- odoo
---

# 准备
- **odoo源代码**，这个是必须的，当前使用[odoo 13](https://github.com/odoo/odoo),我采用的是直接下载13.0源代码的方式，这可以忽略odoo的git history，但后期通过pull源码打补丁也就有点复杂，这个在后面再考虑了
- **python 3.7.6**，最开始我用的windows app store上的python，貌似有权限问题，查了一下，从官网重新下的python，我当时看到的最新的python版本是3.8+，但这个后来通过实践，应该是没法支持odoo 13，还是老实地用3.7.6
- **vscode**，python我不是很熟，IDEA作为编辑器觉得有些重量级了，拿了vscode，先试试再说。记得要装python的插件
- **Odoo development cookbook**, 这个算是入门书，比较合适，千万别看国内的翻译版，有些概念直译很难懂；我在[Packpub](https://www.packtpub.com/)上买的，不算贵，$5；当前好像还没有13版的，就拿了12版的看了看，到目前为止，没有因为版本差异导致学不下来的；epub用MS edge还是很好的
- **docker**，这应该不用说了，开发基本都要的，虽然这里只在上面起了一个postgres container，还是很方便

# 当前环境

我用的windows 10，部分脚本没办法从cookbook上直接拷来用，有些要改成windows batch，但基本上改动都不会太复杂。

# 配置
1. odoo的启动方式有很多种，包括直接安装到本机、从docker启动等等，官方的文档和cookbook里都有介绍。我们这里准备做开发，当前有很多东西不是很清楚，为了能够直接的查看odoo源码，了解odoo的设计机制，我们这里采用odoo source code + postgres container的方式带起来，此处可参考cookbook。
- 通过init脚本可以生成配置文件
```
    %~dp0/../env/Scripts/python.exe %~dp0/../src/odoo/odoo-bin ^
    -c %~dp0/../odooae.cfg --stop-after-init ^
    -d odoo ^
    -i base ^
    --save ^
    --data-dir %~dp0/../filestore ^
    --addons-path %~dp0/../src/odoo/odoo/addons,%~dp0/../src/odoo/addons,%~dp0/../local,%~dp0/../src/partner-contact ^
    --db_host=localhost ^
    --db_port=8070 ^
    --db_user=odoo ^
    --db_password=odoo ^
```
- 通过start脚本可以启动odoo
```
%~dp0/../env/Scripts/python.exe %~dp0/../src/odoo/odoo-bin -c %~dp0/../odooae.cfg
```
2. cookbook也推荐了python virtualenv，通过virtualenv可以较为轻松的创建独立的python环境，以满足不同项目的不同需求。
3. 为了保证vscode对python的自动补全功能，需要配置vscode项目参数，路径为./vscode/settings.json，我这里将odoo源码路径加了进去。
```
{
    "python.pythonPath": "env\\Scripts\\python.exe",
    "python.autoComplete.extraPaths": ["D:\\6.faststep\\odoo\\odoo-dev\\odooae\\src\\odoo"]
}
```
4. 我们在这里用使用git管理源代码，通过配置，我们应当只track自己的项目代码，基本上是odoo module代码，而odoo的代码或者第三方插件的代码都不做跟踪，如果考虑方便后期直接打补丁，那么可以直接git clone这些代码。
