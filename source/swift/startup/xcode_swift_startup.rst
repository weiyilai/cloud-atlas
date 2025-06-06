.. _xcode_swift_startup:

=====================
Xcode Swift Startup
=====================

Xcode和Apple ID
==================

Xcode启动后，需要添加一个Apple ID用于后续程序开发或发布:

- ``Xcode => Settings...`` ，选择 ``Accounts`` 面板
- 点击 ``+`` 增加自己的Apple ID账号

  - 如果已经注册过 Apple Developer program ，则选项中有 Apple Distribution certificate
  - 没有注册过，则添加 Apple Development certificate

Playgrounds
============

``File => New => Playground`` : 交互环境，用于快速开发和评估Swift代码。

在Playground中，不需要编译和运行完整项目；相反，playgrounds直接评估swift代码，用于测试，是一个轻量级的工作环境。

函数
======

Swift类似C，打印使用 ``print()`` 函数 (函数编程)将变量输出到控制台

注释和Markup格式
===================

注释类似 C++ :

- ``//`` 使用快捷键 ``command-/`` 可以来回切换 ``//`` 注释是否生效
- ``/* ... */`` (多行注释)

一个有用的Markup格式注释可以方便我们阅读:

- 修改多行注释 ``/* ... */`` 的开头那行 ``/*`` ，改为 ``/*:`` ，则这段注释就是 **Markup** 格式
- 选择Xcode菜单 ``Editor -> Show Rendered Markup`` ，则在源代码显示中，Markup格式注释会自动渲染，方便阅读

:ref:`git` 平台集成
======================

`Putting Your Xcode Project on GitHub, Bitbucket, or GitLab <https://swiftdevjournal.com/putting-your-xcode-project-on-github-bitbucket-or-gitlab/>`_ :

- 支持和 :ref:`gitlab` 集成，可以部署自建帮本管理系统
