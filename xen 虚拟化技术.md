## xen 虚拟化技术

    第50天 【xen虚拟化技术基础(01)】

### 回顾:
---
- 虚拟化技术的分类：
    + 模拟：Emulation，模拟出的硬件架构体系与物理机底层架构体系不同。
        * Qemu, PearPC, Bochs
    + 完全虚拟化：Full Virtualization，Native Virtualization，提供的硬件架构与宿主机相同，或者兼容。
        * HVM
        * VMware Workstation, VirtualBox, VMWare Server, Parallels Desktop, KVM, XEN
    + 半虚拟化：ParaVirtualization
        * GuestOS：知晓自己是运行在 Virtualization 环境。
        * Hypercall
        * Xen, UML(User-Mode Linux)
    + OS级别的虚拟化：
        * Guest直接运行在用户空间
        * 容器级虚拟化
        * 将用户空间分隔为多个，彼此间互相隔离
        * OpenVZ, LXC(Linux Container), libcontainer, Virtuozzo, Linux V Server
    + 库级别虚拟化
        * WINE
    + 程序级虚拟化
        * jvm

    + Type-I：hypervisor 直接运行在硬件上。xen，VM ESXi
    + Type-II
    + IaaS：Infrastructure as a Service
    + PaaS：Platfrom as a Service

### Xen:
---
剑桥大学研发，开源 VMM；

- Xen 组成部分：
    + Xen Hypervisor
        * 分配 CPU、Memory、Interrupt
    + Dom0
        * 特权域，I/O 分配
            - 网络设备
                + net-front(GuestOS), net-backend
            - 块设备
                + block-front(GuestOS), block-backend
        * Linux Kernel：
            - 2.6.37：开始支持运行 Dom0
            - 3.0：对关键特性进行优化
        * 提供管理 DomU 工具栈
            - 用于实现对虚拟机进行添加、启动、快照、停止、删除等操作；
    + DomU
        * 非特权域，根据其虚拟化方式实现，有多种类型
            - PV
            - HVM
            - PV on HVM

- Xen 的 PV 技术：
    + 不依赖于 CPU 的 HVM 特性，但要求 GuestOS 的内核作出修改以知晓自己运行于 PV 环境；
    + 运行于 DomU 中的 OS：Linux(2.6.24+), NetBSD, FreeBSD, OpenSolaris

- Xen 的 HVM 技术：
    + 依赖于 Intel VT 或 AMD AMD-V，还要依赖于 Qemu 来模拟 IO 设备；
    + 运行于 DomU 中的 OS：几乎所有支持此 X86 平台的；

- PV on HVM：
    + CPU 为 HVM 模式运行
    + IO 设备为 PV 模式运行
    + 运行于 DomU 中的 OS：只要 OS 能驱动 PV 接口类型的 IO 设备；
        * net-front, blk-front
   
- Xen 的工具栈
    + xm/xend：在 Xen Hypervisor 的 Dom0 中要启动 xend 服务
        * xm：命令行管理工具，有诸多子命令：
            - create
            - destroy
            - stop
            - pause
    + xl

