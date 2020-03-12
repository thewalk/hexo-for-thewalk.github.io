---
title: '[odoo]开发环境配置'
date: 2020-02-25 10:30:16
tags: 
- odoo
- tech
category:
- odoo
---

# 准备
- **odoo源代码**，这个是必须的，当前使用[odoo 13](https://github.com/odoo/odoo),我采用的是直接下载13.0源代码的方式，这可以忽略odoo的git history，但后期通过pull源码打补丁也就有点复杂，这个在后面再考虑了
- **python 3.7.6**，最开始我用的windows app store上的python，貌似有权限问题，查了一下，从官网重新下的python，我当时看到的最新的python版本是3.8+，但这个后来通过实践，应该是没法支持odoo 13，还是老实地用3.7.6
- **vscode**，python我不是很熟，IDEA作为编辑器觉得有些重量级了，拿了vscode，先试试再说。记得要装python的插件。
- **Odoo development cookbook**, 这个算是入门书，比较合适，千万别看国内的翻译版，有些概念直译很难懂；我在[Packpub](https://www.packtpub.com/)上买的，不算贵，$5；当前好像还没有13版的，就拿了12版的看了看，到目前为止，没有因为版本差异导致学不下来的；epub用MS edge还是很好的
- **docker**，这应该不用说了，开发基本都要的，虽然这里只在上面起了一个postgres container，还是很方便

# 配置启动
1. odoo的启动方式有很多种，包括直接安装到本机、从docker启动等等，官方的文档和cookbook里都有介绍。我们这里准备做开发，当前有很多东西不是很清楚，为了能够直接的查看odoo源码，了解odoo的设计机制，我们这里采用odoo source code + postgres container的方式带起来，此处可参考cookbook。（*我用的windows 10，部分脚本没办法从cookbook上直接拷来用，有些要改成windows batch，但基本上改动都不会太复杂。*）
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
3. 为了保证vscode对python的自动补全功能，需要配置vscode项目参数，路径为./vscode/settings.json，我这里将odoo源码路径加了进去，其他的module路径可随时自己添加,此外vscode对python的intellisense并不是很好，换用jedi，比较方便。
    ```
    {
        "python.pythonPath": "${workspaceRoot}\\env\\Scripts\\python.exe",
        "python.autoComplete.extraPaths": [
            "${workspaceRoot}\\src\\odoo",
            "${workspaceRoot}\\src\\odoo\\odoo",
            "${workspaceRoot}\\src\\odoo\\odoo\\addons",
            "${workspaceRoot}\\src\\odoo\\addons",
        ],
        "python.jediEnabled": true,
        "files.exclude":{
            "**/*.pyc": true
        }
    }
    ```
4. 我们在这里用使用git管理源代码，通过配置，我们应当只track自己的项目代码，基本上是odoo module代码，而odoo的代码或者第三方插件的代码都不做跟踪，如果考虑方便后期直接打补丁，那么可以直接git clone这些代码。

# 调试代码
odoo在界面端增加了比较多的调试能力，但是还不足够，相比较java的调试来讲，还是差太多。我们这里想要搭建一个方便调试的环境，cookbook中提到了很多，包括使用pdb，不过没有界面，全都是控制台命令，太low了，所以想换成IDE支持的。我先后查找实践了包括pycharm、vscode这两样IDE的配置方式，pycharm自己只能搭起来run的脚本，不知道怎么去debug；vscode也是困难重重，使用launch的方式尝试带起来odoo，但最后总是挂掉，换了attach方式才解决问题。

## 基础

首先确保认真读过cookbook中关于debug的一章，都是很有用的，只不过pdb不够高效。
- auto-reload, 通过安装watchdog，配置启动参数dev，达到变更代码后，能够自动热更新，这个相比较spring、angular都要强很多（或许我还没研究过高级功能，spring boot是要手动重启的；angular虽然能自动build，但是build的是全部模块，odoo这build的只是局部模块，速度应该是快了很多）。
    ```
    pip install watchdog
    odoo/odoo-bin -c ~/odoo-dev/my-instance.cfg --dev=all
    ```
    以上是从cookbook摘出来的相关命令，但是为了高级debug，我们把pdb从dev参数中去掉,batch命令根据以下修改。
    ```
    odoo/odoo-bin -c ~/odoo-dev/my-instance.cfg --dev=reload,qweb,werkzeug,xml
    ```
- logs，不多说了。
- shell，这个本来应该配合pdb使用，比较麻烦，后面应该不需要，需要的话再说
- pdb，这个不想用
- OCA matainer quality tools主要说的就是travisCI，别的项目用过，不怎么好用，有空研究下azure devops相应的做法
- pylint，可以在vscode setting里加，属于IDE控制范畴，如果在项目上安装，可以参照cookbook，主要是语法约束
- flake8，也是一个语法约束器
- odoo debug menu，这个看cookbook就行，还没试，应该比较厉害

## 使用vscode进行debug

vscode的debug配置通过修改./vscode/launch.json来完成，详细的介绍可以通过[官方文档](https://code.visualstudio.com/docs/editor/debugging#_launch-configurations)来了解;主要用到的调试工具是ptvsd，全称是[Python Tools for Visual Studio debug server](https://github.com/microsoft/ptvsd)。

### 使用launch方式（失败）

原本希望通过launch的方式进行debug，配置如下,debug模式能起来，但是总是到```src\odoo\odoo\tools\config.py```这个运行过程就会raise exception，odoo最后也没有带起来。
```
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "odoo",
      "type": "python",
      "request": "launch",
      "stopOnEntry": false,
      "console": "integratedTerminal",
      "program": "${workspaceRoot}\\src\\odoo\\odoo-bin",
      "args": ["--config", "${workspaceRoot}\\odooae.cfg"],
      "cwd": "${workspaceRoot}",
      "pythonPath": "${config:python.pythonPath}"
    }
  ]
}
```
通过查看exception上下文，感觉像是odoo还没有读到自定义的配置文件，就直接挂了，具体的原因这里没有推断出来，不做过多阐述了。

### 使用attach方式

通过摸索launch方式，感觉ptvsd介入的过早了，可能导致odoo起来就挂掉，所以按attach方式，手动设置debug切入点，具体使用可以查看官网。

1. 我直接在配置完成点（即```src\odoo\odoo\tools\config.py```尾部）放置debug切入点，如下。
    ```
    config = configmanager()

    # debug inception
    import ptvsd
    ptvsd.enable_attach()
    ptvsd.wait_for_attach() 
    ```
1. 相应的luanch.json需要做修改
    ```
    {
        // Use IntelliSense to learn about possible attributes.
        // Hover to view descriptions of existing attributes.
        // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
            "name": "odoo",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "127.0.0.1",
            "pathMappings": [
                {
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "${workspaceFolder}"
                }
            ]
            }
        ]
    }
    ```
1. 接下来可以debug了，还是通过启动脚本带起来odoo，不过，由于debug切入点的配置，odoo会停在那，等着ptvsd的接入。
1. 最后我们点击vscode的debug或者F5，通过ptvsd接进去就可以了，然后就可以愉快地调试了。