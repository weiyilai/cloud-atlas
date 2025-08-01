.. _nvidia_p4_pi_docker:

====================================================
树莓派安装NVIDIA P4 GPU运行 ``nvidia-docker`` 容器
====================================================

.. note::

   我使用 :ref:`pi_5` + :ref:`tesla_p4` 来构建一个7x24低功耗(无声)持续运行的机器学习环境

   实践将结合我之前的经验:

   - :ref:`install_nvidia_linux_driver`
   - :ref:`install_nvidia_cuda`
   - :ref:`nvidia-docker`

.. note::

   由于 :ref:`nvidia-driver_pi_os` 遇到困难挫折，所以我推翻重新采用 :ref:`ubuntu_linux` for Raspberry Pi作为操作系统，再次尝试安装 ``nvidia-driver``

实践环境
=============

- :ref:`pi_5` 安装了 :strike:`Respberry Pi OS` Ubuntu 24.04.02 LTS ，即 :ref:`debian` 12 (bookworm)，这是一个 :ref:`arm` 架构的低功耗微型计算机，所以后续安装 :ref:`install_nvidia_linux_driver` 需要选择 ``aarch64`` 架构
- :ref:`tesla_p4` 加装了淘宝购买的散热器，并通过 ``OCuLink`` Dock连接到 :ref:`pi_5`

:ref:`tesla_p4` 加电后再启动连接的 :ref:`pi_5` ，进入host主机系统后执行 ``lspci`` 命令可以看到识别出 :ref:`tesla_p4` :

.. literalinclude:: nvidia_p4_pi_docker/lspci
   :caption: 启动系统后检查识别的 :ref:`tesla_p4`
   :emphasize-lines: 5

架构
======

.. image:: ../../_static/docker/nvidia_container_runtime_for_docker.png

CUDA软件堆栈
---------------

NVIDIA将GPU驱动和开发组件(Toolkits)分别组合成:

- ``cuda``
- ``cuda-drivers`` : ``cuda`` 的子集

由于虚拟化和容器化技术的发展，我们可以在不同的层次分别安装:

- 物理主机: ``cuda-drivers``
- 虚拟机(PassThrough GPU):

  - 直接在虚拟机内运行应用及开发: ``cuda``
  - 虚拟机内通过容器运行应用及开发:

    - 虚拟机: ``cuda-drivers``
    - 容器: ``cuda``

- (裸金属)容器: ``cuda``

简而言之:

- 只有实际运行应用及开发的 ``主机/虚拟化/容器`` 层才需要完整安装 ``cuda``
- 其他作为支持层的层只需要安装 ``cuda-dirvers``

Host主机安装 ``nvidia-driver``
==================================

如上文所述，在 :ref:`pi_5` Host主机上我规划部署 :ref:`docker` (作为 :ref:`kubernetes` 主机节点)，所以只需要安装 ``cuda-drivers`` 

.. note::

   在 :ref:`install_nvidia_linux_driver` 我曾经采用过两种方式安装 ``cuda-drivers`` :

   - 手工下载安装 `NVIDIA官方提供的 P40 驱动 <https://www.nvidia.com/download/index.aspx#>`_ (我在 :ref:`nvidia-driver_pi_os` 也尝试了手工安装驱动方法，但是在Raspberry Pi OS上没有成功
   - 通过Linux发行版软件仓库方式安装NVDIA CUDA驱动

   本次实践我采用后者 **软件仓库方式**

准备工作
-----------

按照 :ref:`install_cuda_prepare` 检查和准备:

- 由于 :ref:`pi_5` 只有8GB内存，所以不建议启用 :ref:`above_4g_decoding` (应该也没有这个BIOS设置选项)
- 验证系统已经安装gcc以及对应版本:

.. literalinclude:: nvidia_p4_pi_docker/gcc
   :caption: 检查系统安装的gcc版本

NVIDIA驱动需要主机已经安装了 Kernel headers 和 开发软件包

如果系统尚未安装 ``gcc`` 可以采用 :ref:`debian_init` 纯后台服务器系统安装开发工具的方式(安装 ``build-essential`` 为主):

.. literalinclude:: ../../linux/debian/debian_init/debian_init_vimrc_dev
   :caption: 安装纯后台开发工具

- CUDA驱动需要内核头文件以及开发工具包来完成内核相关的驱动安装，因为内核驱动需要根据内核进行编译

安装 **linux-headers** :

.. literalinclude:: nvidia_p4_pi_docker/linux-headers
   :caption: 安装inux-headers

CUDA软件仓库
--------------

从NVIDIA官方提供 `NVIDIA CUDA Toolkit repo 下载 <https://developer.nvidia.com/cuda-downloads>`_

- 由于是 :ref:`pi_5` ARM架构，我选择了 ``Linux >> arm64-sbsa (Server Base System Architecture) >> Native >> Ubuntu >> 22.04 >> deb (network)``

  - Compilation 步骤可选 ``Native`` (只编译相同架构的代码)和 ``Cross`` (可编译不同架构代码)，我选择 ``Native``
  - Ubuntu版本选择 ``22.04`` 对应的是 :ref:`debian` 12 (bookworm)，如果选 Ubuntu 24.04 则对应的是debian 13

- 安装步骤:

.. literalinclude:: nvidia_p4_pi_docker/cuda_install
   :caption: 安装仓库

仓库安装 ``cuda-drivers``
----------------------------

.. note::

   使用软件仓库网络安装 ``cuda-drivers`` 需要主机安装好对应的 ``linux-headers``

.. literalinclude:: ../../machine_learning/hardware/nvidia_gpu/install_nvidia_linux_driver/cuda_driver_debian_ubuntu_repo_install
   :language: bash
   :caption: Debian/Ubuntu使用NVIDIA官方软件仓库安装CUDA驱动

安装过程会使用 :ref:`dkms` 编译NVIDIA内核模块，果然在NVIDIA官方支持的Ubuntu平台编译安装非常顺利，没有遇到 :ref:`nvidia-driver_pi_os` 中痛苦的折磨。

然而...

GPU初始化异常排查
===================

虽然驱动安装正常，重启系统后却发现 ``nvidia-smi`` 执行显示驱动没有加载。WHY

- 检查 ``dmesg -T`` 输出，发现原因是总线没有响应初始化命令导致驱动加载失败:

.. literalinclude:: nvidia_p4_pi_docker/gpu_init_fail
   :caption: GPU初始化失败

.. note::

   目前初步判断可能存在问题:

   - 使用 OCuLink 连接存在硬件问题或使用问题
   - 购买的 :ref:`tesla_p4` 可能有硬件隐患

   我准备采用替换方法排查，后续来尝试解决这个问题

Host主机安装NVIDIA Container Toolkit
======================================

- 配置软件生产级仓库:

.. literalinclude:: nvidia_p4_pi_docker/nvidia-container-toolkit_repository
   :caption: 配置 NVIDIA Container Toolkit 仓库

另外一种仓库是实验性仓库，通过修订 ``/etc/apt/sources.list.d/nvidia-container-toolkit.list`` 配置(我没有使用):

.. literalinclude:: nvidia_p4_pi_docker/nvidia-container-toolkit_repository_experimental
   :caption: 可选配置实验性仓库

- 更新仓库的软件包列表:

.. literalinclude:: nvidia_p4_pi_docker/apt_update
   :caption: 更新仓库软件列表

- 安装 NVIDIA Container Toolkit:

.. literalinclude:: nvidia_p4_pi_docker/apt_install
   :caption: 安装NVIDIA Container Toolkit

配置
=========

.. note::

   确保已经完成以下软件包:

   - 受支持的容器引擎(Docker, Containerd, CRI-O, Podman)
   - 已安装 NVIDIA Container Toolkit

配置Docker
--------------

- 使用 ``nvidia-ctk`` 命令配置容器运行时:

.. literalinclude:: nvidia_p4_pi_docker/nvidia-ctk
   :caption: ``nvidia-ctk`` 命令配置容器运行时

``nvidia-ctk`` 会修订 ``/etc/docker/daemon.json`` ，这样Docker就会使用 NVIDIA Container Runtime

- 重启 Docker 服务:

.. literalinclude:: nvidia_p4_pi_docker/restart_docker
   :caption: 重启docker服务

配置containerd(为kubernetes)
-------------------------------

对于使用 ``containerd`` 的Kubernetes环境，使用以下命令配置运行时:

.. literalinclude:: nvidia_p4_pi_docker/nvidia-ctk_containerd
   :caption: 配置containerd

此时 ``nvidia-ctk`` 命令会修订 ``/etc/containerd/config.toml`` 配置

参考
======

- `Using NVIDIA GPU within Docker Containers <https://marmelab.com/blog/2018/03/21/using-nvidia-gpu-within-docker-container.html>`_ (在安装 ``NVIDIA Container Toolkit`` 之前，先参考 `CUDA Installation Guide for Linux <https://docs.nvidia.com/cuda/cuda-installation-guide-linux/>`_ 完成 ``cuda-driver`` 安装)
- `Enabling GPUs in the Container Runtime Ecosystem <https://devblogs.nvidia.com/gpu-containers-runtime/>`_
- **已废弃** `Build and run Docker containers leveraging NVIDIA GPUs <https://github.com/NVIDIA/nvidia-docker>`_
- `Installing the NVIDIA Container Toolkit <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html>`_ 从2023年8月开始， ``nvidia-docker`` 已经被 ``NVIDIA Container Toolkit`` 替代，所以本文实践部署替代了之前的 :ref:`nvidia-docker`

  - 前置步骤是驱动安装，参考 `CUDA Installation Guide for Linux#Driver Installation <https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#driver-installation>`_ : `NVIDIA Driver Installation Guide <https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html>`_

- `JeffGeerling的网站上 Raspberry Pi PCIe Database#GPUs (Graphics Cards) <https://pipci.jeffgeerling.com/#gpus-graphics-cards>`_
