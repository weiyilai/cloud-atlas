.. _why_k8s_dev:

=========================================
为什么要做Kubernetes开发
=========================================

.. _think_kfs:

``最初的野望`` : 为什么要"Kubernetes From Scratch (KFS)"
=========================================================

.. note::

   这是我最初写下的文字，记录我当时的想法。不过我现在改变了目标，我将尝试结合 :ref:`blfs_k8s` 来实践我最初的 "Kubernetes From Scratch (KFS)" 构想。

   以下这段我仅作为回顾

我在雷阵雨之后的路上行走的时候，天空中云团浮动，时时有半明半暗的的阳光洒下。

我突然想到最近几天在部署的 :ref:`bootstrap_kubernetes_ha` ，依然有很多步骤只是使用诸如 :ref:`kubeadm` 按照手册完成，无法明确底层实现的原理(例如，存储到 :ref:`etcd` 的数据结构，服务相互的接口调用和依赖...)。

就像你安装 :ref:`ubuntu_linux` 或者 :ref:`fedora` ，按照安装引导一步步Next当然没有错，但是你有没有思考过每个步骤底下的调用、数据，这些原理不明晰，那么就如 `禅与摩托车维修艺术 <https://book.douban.com/subject/6811366/>`_ 中所说的 "无法理解机械运作原理而对摩托样的现代设备产生拒绝和厌恶" 。

联想到 :ref:`lfs` ，从源代码编译安装的Linux发行版，每个软件包都是从源代码开始编译配置和安装，从而对Linux软硬件的透彻理解。我也想到，对Kubernetes这样现代容器调度系统的完整构建:

- 尽可能使用低级工具来完成步骤，不追求方便，而是透析Kubernetes庞大复杂的组建底层所使用的技术。例如，密钥认证，会使用 :ref:`tls` (openssl) 来构建，而不是精心包装的上层工具 ``cfssl``
- 不使用快捷工具 :ref:`kubeadm` ，而是使用命令行完成 :ref:`container_runtimes` 配置、镜像下载、镜像运行，手工修订 :ref:`etcd` 数据，完成bootlaunch
- 解析原理，解读代码: 分析设计结构、代码逻辑，进而进行定制修改
- 至少对核心组件进行构建和分析(K8S生态太复杂，无法穷尽)
- 对性能进行优化(内核、网络、存储)，从底层向上优化Kubernetes
- (可选)从 :ref:`lfs` 开始构建轻量级运行 :ref:`kubernetes` 的整个虚拟化云计算环境

``现在的野望`` : 从开发中学习Kubernetes
=========================================

回过头来看我从阿里云转岗到蚂蚁金服的近五年时间，都在围绕Kubernetes工作。虽然也同步不断的学习和实践，有了运维相关的经验，但是渐渐觉得，如果不能在开发上获得突破，那么天花板始终在那里，很难再进一步了。

当然，也不是说Kubernetes运维不好，而是这个系统实在台庞大复杂了，层出不穷的组件和定制，你再怎么熟悉系统也解决不了底层的问题。唯有从源代码开始挖掘，才能一法通万法通。

所以，2023年最后一个月，也借着我的下一阶段开启，我决心做一个 "牧码人"
