.. _dl_env:

====================
深度学习环境
====================

.. note::

   本文学习实践以 `动手学深度学习 v2 - 从零开始介绍深度学习算法和代码实现 <https://space.bilibili.com/1567748478/channel/seriesdetail?sid=358497>`_ 为参考

我的运行环境
===============

- 部署 :ref:`linux_jail` (必须同时配置 :ref:`vnet_jail` VNET网络堆栈，否则 :ref:`jupyter` 运行会因为Linux兼容层缺乏 socket 支持而失败) ，启动名为 ``d2l`` 的Jail容器
- 为方便工作，完成 :ref:`linux_jail_init` ，通过ssh登录容器以后 ``chroot`` 进入 :ref:`debian` 运行环境

可能需要的依赖编译环境，可以安装 ``build-essential`` 软件包精简安装( :ref:`debian_init` ):

.. literalinclude:: ../../linux/debian/debian_init/debian_init_vimrc_dev
   :caption: 安装vim以及服务器开发所需软件集

`动手学深度学习 v2 - 从零开始介绍深度学习算法和代码实现 <https://space.bilibili.com/1567748478/channel/seriesdetail?sid=358497>`_ 教程中李沐安装的是python 3.8，我这里替换为 python-dev (包含了 python) 

安装Coda
==============

- :ref:`install_conda` ( 也可以 :ref:`install_anaconda` )

- `Anaconda Download <https://www.anaconda.com/download/success>`_ 提供了Anaconda和Miniconda安裝下載(腳本)

.. literalinclude:: ../startup/install_conda/install_miniconda
   :caption: 安裝miniconda

- 在同意了license之后，按照提示进行安装

.. literalinclude:: ../startup/install_conda/install_miniconda_interact
   :caption: 交互方式安装
   :emphasize-lines: 11,26

安装 ``d2l`` 环境
=====================

- 使用 ``conda`` 来安装和激活 ``d2l-zh`` 环境:

.. literalinclude:: dl_env/create_d2l
   :caption: 创建 ``d2l-zh`` 环境

创建/销毁 环境的时候需要使用 ``-n`` 参数来指定名字

以上命令创建环境时提示信息:

.. literalinclude:: dl_env/create_d2l_output
   :caption: 创建 ``d2l-zh`` 环境时提示信息

- 在激活(activate) ``d2l-zh`` 环境之后，可以看到 :ref:`virtualenv` 的提示符从 ``(base)`` 变成了 ``(d2l-zh)`` ，此时检查 ``python`` 和 ``pip`` 就会看到都在 ``~/conda/envs/d2l-zh/bin/`` 目录下( ``conda`` 会在自己的 ``envs`` 目录下构建不同的Python运行环境，也是管理Python环境的好样板):

.. literalinclude:: dl_env/d2l-zh
   :caption: 检查 ``d2l-zh`` 工作环境

- 安装需要的软件包:

.. literalinclude:: dl_env/d2l-zh_install
   :caption: 在 ``d2l-zh`` :ref:`virtualenv` 环境中继续安装必要软件包

在安装软件包的时候出现如下报错

.. literalinclude:: dl_env/d2l-zh_install_error
   :caption: 在 ``d2l-zh`` 环境安装软件包报错
   :emphasize-lines: 22

参考 `AttributeError: module 'pkgutil' has no attribute 'ImpImporter'. Did you mean: 'zipimporter'? <https://stackoverflow.com/questions/77364550/attributeerror-module-pkgutil-has-no-attribute-impimporter-did-you-mean>`_ ，原因是移除了一个长期不使用的 ``pkgutil.ImpImporter`` 类， ``pip`` 命令可能不能和 ``Python 3.12`` 以期工作。解决的方法是手工在 ``Python 3.12`` 中安装pip:

.. literalinclude:: dl_env/upgrade_pip
   :caption: 手工安装pip

不过，上述更新pip没有解决问题，原因还是安装 ``numpy`` 的问题: `NumPy does not install on Python 3.12.0b1 #23808 <https://github.com/numpy/numpy/issues/23808>`_ 说明需要升级到 ``NumPy 1.26.0`` 才解决问题。所以实际上需要手工指定安装 ``NumPy`` 版本。原先安装失败是因为 ``jupyter`` 已经是比较古早的notebook了，依赖了比较早期的 ``numpy`` 版本。

我的解决方法是使用 ``jupyterlab`` (下一代jupyter)来替代 ``jupyter`` ，这样就会自动安装搞版本 `numpy`` ，也就解决了 ``Python 3.12`` 上运行的困境。

下载代码和执行
================

一切就绪，现在下载代码和执行:

.. literalinclude:: dl_env/d2l-zh_run
   :caption: 在 ``d2l-zh`` 下载代码和执行

如果一切正常(我在 :ref:`freebsd_jail` 环境中折腾了很久，终于通过 :ref:`linux_jail` + :ref:`vnet_jail` 运行成功)，就可以看到 ``jupyter`` 启动后监听在 ``8888`` 端口，就可以通过浏览器访问 http://127.0.0.1:8888

.. note::

   由于我是在 Jail 环境中运行 Jupyter Notebook，所以需要设置 :ref:`jupyter_remote`

异常排查
===========

我在学习 `B站: 动手学深度学习 v2 > 03 安装 <https://www.bilibili.com/video/BV18p4y1h7Dr?spm_id_from=333.788.player.player_end_recommend_autoplay&vd_source=9e81a12fc8eb4223ba7650a40a5ce9a7>`_ ，参考李沐课程，加载某个 ``.ipynb`` 文件时页面会提示错误: ``Error Starting Kernel! NetworkError when attempting to fetch resource.``

检查后端 ``jupyter`` 终端输出信息可以看到出现异常时:

.. literalinclude:: dl_env/thread
   :caption: 加载 ``.jpynb`` 后台异常

此时在系统dmesg日志中会添加一条:

.. literalinclude:: dl_env/dmesg
   :caption: ``jupyter`` 进程被杀死的系统日志

在google的AI提示: 如果在jail内部创建一个线程遇到这个错误，可能需要调整jail配置允许必要的系统调用。不过，AI的建议是很泛泛的，只是针对这个现象而不是具体的报错。

AI举了一个例子，例如允许 clone() 系统调用，则配置 ``allow.sysvipc = 1;`` (这个配置在Jail中运行 :ref:`pgsql` 需要，不过现在应该使用分离的3个配置(我这里记录备用):

 .. literalinclude:: dl_env/pgsql_jail.conf
    :caption: 在Jail中运行 :ref:`pgsql` 需要配置




参考
======

- `B站: 动手学深度学习 v2 > 03 安装 <https://www.bilibili.com/video/BV18p4y1h7Dr?spm_id_from=333.788.player.player_end_recommend_autoplay&vd_source=9e81a12fc8eb4223ba7650a40a5ce9a7>`_
