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

        * Xen 的 PV 技术：
            - 不依赖于 CPU 的 HVM 特性，但要求 GuestOS 的内核作出修改以知晓自己运行于 PV 环境；
            - 运行于 DomU 中的 OS：Linux(2.6.24+), NetBSD, FreeBSD, OpenSolaris

        * Xen 的 HVM 技术：
            - 依赖于 Intel VT 或 AMD AMD-V，还要依赖于 Qemu 来模拟 IO 设备；
            - 运行于 DomU 中的 OS：几乎所有支持此 X86 平台的；

        * PV on HVM：
            - CPU 为 HVM 模式运行
            - IO 设备为 PV 模式运行
            - 运行于 DomU 中的 OS：只要 OS 能驱动 PV 接口类型的 IO 设备；
                + net-front, blk-front


- Xen 的工具栈  
    + xm/xend：在 Xen Hypervisor 的 Dom0 中要启动 xend 服务   
        * xm：命令行管理工具，有诸多子命令：
            - create
            - destroy
            - stop
            - pause
    + xl：基于 libxenlight 提供的轻量级的命令行工具栈；
    + xe/xapi：提供了对 xen 管理的 api， 因此多用于 cloud 环境；Xen Server, XCP
    + virsh/libvirt：Libvirt 是一组软件的汇集，提供了管理虚拟机和其它虚拟化功能（如：存储和网络接口等）的便利途径。这些软件包括：一个长期稳定的 C 语言 API、一个守护进程（libvirtd）和一个命令行工具（virsh）。Libvirt 的主要目标是提供一个单一途径以管理多种不同虚拟化方案以及虚拟化主机，包括：KVM/QEMU，Xen，LXC，OpenVZ 或 VirtualBox hypervisors.

- XenStore：
    + 为各 Domain 提供的共享信息存储空间；有着层级机构的名称空间；位于 Dom0。

- CentOS 对 Xen 的支持：
    + RHEL 5.7-：默认的虚拟化技术为 Xen；
        * kernel version：2.6.18
            - kernel-
            - kernel-xen
    + RHEL 6+：仅支持kvm
        * Dom0：不支持
        * DomU：支持

- 如何在 CentOS 6.6 上使用 Xen；
    1. 编译 3.0 以上版本的内核，启动对 Dom0 的支持；
    2. 编译 Xen 程序；
    3. 制作好的相关程序包的项目：
        + xen4centos
        + xen made easy

- 在 CentOS 6.9 上使用 yum 源安装 Xen
    + 查看当前 CPU 是否支持虚拟化：
    ```
        # grep -E '(vmx|svm)' /proc/cpuinfo
    ```
    + 当前系统内核版本：
    ```
        # uname -r
        2.6.32-696.el6.x86_64
    ```
    + 设置 yum 源，只用阿里云的镜像
    ```
        # vim xen.repo
        [xenCentos]
        name=Xen for CentOS 6
        baseurl=https://mirrors.aliyun.com/centos/6.9/virt/x86_64/xen/
        gpgcheck=0
    ```
    + 使用 yum 安装
    ```
        # yum install xen -y
        主要安装了一下软件包：
        Dependencies Resolved

        =================================================================================================================================
         Package                               Arch                Version                                  Repository              Size
        =================================================================================================================================
        Installing:
         linux-firmware                        noarch              20170828-77.gitb78acc9.el6               xenCentos               55 M
             replacing  ivtv-firmware.noarch 2:20080701-20.2
             replacing  kernel-firmware.noarch 2.6.32-573.3.1.el6
             replacing  ql2100-firmware.noarch 1.19.38-3.1.el6
             replacing  ql2200-firmware.noarch 2.02.08-3.1.el6
             replacing  ql23xx-firmware.noarch 3.03.27-3.1.el6
             replacing  ql2400-firmware.noarch 7.03.00-1.el6_5
             replacing  ql2500-firmware.noarch 7.03.00-1.el6_5
             replacing  rt61pci-firmware.noarch 1.2-7.el6
             replacing  rt73usb-firmware.noarch 1.8-7.el6
             replacing  xorg-x11-drv-ati-firmware.noarch 7.5.99-3.el6
         xen                                   x86_64              4.4.4-34.el6                             xenCentos              1.0 M
        Installing for dependencies:
         SDL                                   x86_64              1.2.14-7.el6_7.1                         base                   193 k
         bridge-utils                          x86_64              1.2-10.el6                               base                    30 k
         glusterfs                             x86_64              3.7.9-12.el6                             base                   422 k
         glusterfs-api                         x86_64              3.7.9-12.el6                             base                    58 k
         glusterfs-client-xlators              x86_64              3.7.9-12.el6                             base                   1.1 M
         glusterfs-libs                        x86_64              3.7.9-12.el6                             base                   313 k
         kernel                                x86_64              4.9.75-30.el6                            xenCentos               42 M
         kpartx                                x86_64              0.4.9-100.el6_9.1                        updates                 70 k
         libusb1                               x86_64              1.0.9-0.7.rc1.el6                        base                    80 k
         python-lxml                           x86_64              2.2.3-1.1.el6                            base                   2.0 M
         qemu-img                              x86_64              2:0.12.1.2-2.503.el6_9.4                 updates                846 k
         usbredir                              x86_64              0.5.1-3.el6                              base                    41 k
         xen-hypervisor                        x86_64              4.4.4-34.el6                             xenCentos              4.7 M
         xen-libs                              x86_64              4.4.4-34.el6                             xenCentos              432 k
         xen-licenses                          x86_64              4.4.4-34.el6                             xenCentos               88 k
         xen-runtime                           x86_64              4.4.4-34.el6                             xenCentos              9.9 M
         yajl                                  x86_64              1.0.7-3.el6                              base                    27 k

    ```
    + 此时查看 /etc/grub.conf 文件，会看到新安装了一个内核（kernel-4.9.75-30.el6.x86_64)，此内核是运行在 Xen 的 Dom0 中，所以这里不能设置从这里启动。
    ```
        ]# cat /etc/grub.conf
        # grub.conf generated by anaconda
        #
        # Note that you do not have to rerun grub after making changes to this file
        # NOTICE:  You have a /boot partition.  This means that
        #          all kernel and initrd paths are relative to /boot/, eg.
        #          root (hd0,0)
        #          kernel /vmlinuz-version ro root=/dev/mapper/vg_node1-lv_root
        #          initrd /initrd-[generic-]version.img
        #boot=/dev/sda
        default=0
        timeout=5
        splashimage=(hd0,0)/grub/splash.xpm.gz
        hiddenmenu
        title CentOS (4.9.75-30.el6.x86_64)
                root (hd0,0)
                kernel /vmlinuz-4.9.75-30.el6.x86_64 ro root=/dev/mapper/vg_node1-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_LVM_LV=vg_node1/lv_swap rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg_node1/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
                initrd /initramfs-4.9.75-30.el6.x86_64.img
        title CentOS 6 (2.6.32-696.el6.x86_64)
                root (hd0,0)
                kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/vg_node1-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_LVM_LV=vg_node1/lv_swap rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg_node1/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
                initrd /initramfs-2.6.32-696.el6.x86_64.img
    ```
    + 查看 /boot 目录下的文件，会发现其有 xen.gz、xen-4.4.gz和xen-4.4.4-34.el6.gz 3个文件，xen.gz、xen-4.4.gz 文件是 xen-4.4.4-34.el6.gz 文件的软连接，而 xen-4.4.4-34.el6.gz 文件即是 Xen 的内核，其需要直接运行在硬件环境上，故需修改 /etc/grub.conf 文件，如下：
    ```
        grub 配置：

        #boot=/dev/sda
        default=0
        timeout=5
        splashimage=(hd0,0)/grub/splash.xpm.gz
        hiddenmenu
        title CentOS (4.9.75-30.el6.x86_64)
                root (hd0,0)
                kernel /xen.gz dom0_mem=1024M cpufreq=xen dom0_max_vcpus=2 dom0_vcpus_pin
                module /vmlinuz-4.9.75-30.el6.x86_64 ro root=/dev/mapper/vg_node1-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_LVM_LV=vg_node1/lv_swap rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg_node1/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
                module /initramfs-4.9.75-30.el6.x86_64.img
        title CentOS 6 (2.6.32-696.el6.x86_64)
                root (hd0,0)
                kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/vg_node1-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_LVM_LV=vg_node1/lv_swap rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg_node1/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
                initrd /initramfs-2.6.32-696.el6.x86_64.img

        配置说明：
            kernel /xen.gz：配置 kernel 从 xen-4.4.4-34.el6.gz 内核启动。
            dom0_mem=1024：设置 dom0 使用的内存大小。
            cpufreq=xen：cpu 使用频率由 xen 控制。
            dom0_max_vcpus=1：dom0 使用的 vcpu 个数为 1。
            dom0_vcpus_pin：将 dom0 的 vcpu 固定在物理核心上。

            grub.conf boot options: http://xenbits.xen.org/docs/unstable/misc/xen-command-line.html
    ```
    + 官方 Man 手册：<https://wiki.xenproject.org/wiki/Xen_Man_Pages>
    + 重启服务器，查看其内核与虚拟化环境：
    ```
        # shutdown -r now 

        # uname -r
        4.9.75-30.el6.x86_64

        # xl list
        Name                                        ID   Mem VCPUs      State   Time(s)
        Domain-0                                     0  1024     2     r-----      37.2
    ```

```
    第50天 【xen虚拟化技术基础(02)】
```

- Xen 工具栈
    + xm/xend
        * 默认 xend 服务没有启动，如果使用 xm 管理，则需先启动 xend 服务（service xend start)
        * xm 命令已经被废弃了，虽然有提供，但不建议使用。
    + xl
        * `xl list` 显示 Domain 的相关信息
            - Xen 虚拟机状态：
                + r: running
                + b: 阻塞
                + p: 暂停
                + s: 停止
                + c: 崩溃
                + d: dying, 正在关闭的过程中

- 如何创建 Xen pv 模式：
    + 定义 DomU 的配置文件
        * kernel
        * initrd 或 initramfs
        * DomU 内核模块
        * 根文件系统
        * swap 设备
        * 将上述内容定义在 DoumU 的配置文件中
        * 注意：xm 与 xl 启动 DomU 使用的配置文件略有不同；
        * 对于 xl 而言，其创建 DomU 使用的配置指令可通过 `man xl.cfg` 获取
        ```
            - 常用指令：
                + name：域唯一名称
                + builder：指明虚拟机的类型，generic 表示 pv，hvm 表示 HVM
                + vcpus：虚拟 cpu 个数
                + maxcpus：最大虚拟 cpu 个数
                + cpus：vcpu 可运行于其上的物理 cpu 列表
                + memory=MBYTES：表示内存大小
                + maxmem=MBYTES：可以使用的最大内存空间
                + on_poweroff：指明关机时采取的 action：
                    * destroy, restart, preserve
                + on_reboot="ACTION"：指明“重启” DomU 时采取的 action
                + on_crash="ACTION"：虚拟机意外崩溃时采取的 action
                + uuid：DomU 的唯一标识
                + disk=[ "DISK_SPEC_STRING", "DISK_SPEC_STRING", ...]：指明磁盘设备，列表。
                + vif=[ "NET_SPEC_STRING", "NET_SPEC_STRING", ...]：指明网络接口，列表。
                + vfb=[ "VFB_SPEC_STRING", "VFB_SPEC_STRING", ...]：指明 virtual frame buffer，列表。
                + pci=[ "PCI_SPEC_STRING", "PCI_SPEC_STRING", ... ]：pci 设备列表。
            - PV 模式专用指令：
                + kernel="PATHNAME": 内核文件路径，此为 Dom0 中的路径；
                + ramdisk="PATHNAME"：为 kernel 指定内核提供的 ramdisk 文件路径；
                + bootloader="PROGRAM"：如果 DomU 使用自己的 kernel 及 ramdisk, 此时需要一个 Dom0 中的应用程序来实现其 bootloader 功能；
                + root="STRING"：指明根文件系统；
                + extra="STRING"：额外传递给内核引导时使用的参数；
            - 磁盘参数制定方式：
                + 官方文档：http://xenbits.xen.org/docs/unstable/man/xl-disk-configuration.5.html
                + [<target>, [<format>, [<vdev>, [<access>]]]]
                    - <target> 表示磁盘映像文件或设备文件路径：/images/xen/linux.img, /dev/myvg/linux
                    - <format> 表示磁盘格式，如果是映像文件，有多种格式，例如：raw, qcow, qcow2 ...
                    - <vdev> 此设备在 DomU 被识别为硬件设备类型，支持hd[x], xvd[x], sd[x] etc.
                    - <access> 访问权限，
                        ro, r：只读
                        rw, w：读写

                        disk = [ "/images/xen/linux.img,raw,xvda,rw","...","..." ]
                + 使用 qemu-img 管理磁盘映像文件
                    - create [-f fmt] [-o options] filename [size]
                        + 可创建 sparse(稀疏) 格式的磁盘映像文件，是默认格式
                            # mkdir -pv /images/xen
                            mkdir: created directory `/images'
                            mkdir: created directory `/images/xen'
                            两种创建方式：
                            1、
                            # qemu-img create -f raw -o size=2G /images/xen/busybox.img
                            Formatting '/images/xen/busybox.img', fmt=raw size=2147483648
                            2、
                            # qemu-img create -f raw  /images/xen/busybox.img 2G 
                            Formatting '/images/xen/busybox.img', fmt=raw size=2147483648 

                            使用 ll 看是 2G，实则为 0
                            # ll -h /images/xen/busybox.img 
                            -rw-r--r--. 1 root root 2.0G Feb  4 17:03 /images/xen/busybox.img
                            # du -sh /images/xen/busybox.img
                            0       /images/xen/busybox.img

                            格式化：
                            # mke2fs -t ext2 busybox.img 
                            mke2fs 1.41.12 (17-May-2010)
                            busybox.img is not a block special device.
                            Proceed anyway? (y,n) y
                            Filesystem label=
                            OS type: Linux
                            Block size=4096 (log=2)
                            Fragment size=4096 (log=2)
                            Stride=0 blocks, Stripe width=0 blocks
                            131072 inodes, 524288 blocks
                            26214 blocks (5.00%) reserved for the super user
                            First data block=0
                            Maximum filesystem blocks=536870912
                            16 block groups
                            32768 blocks per group, 32768 fragments per group
                            8192 inodes per group
                            Superblock backups stored on blocks: 
                                    32768, 98304, 163840, 229376, 294912

                            Writing inode tables: done                            
                            Writing superblocks and filesystem accounting information: done

                            This filesystem will be automatically checked every 36 mounts or
                            180 days, whichever comes first.  Use tune2fs -c or -i to override.
                            # du -sh busybox.img 
                            33M     busybox.img

                            挂载：
                            # mount -o loop busybox.img /mnt. // mount -o loop 挂载本地回环设备
                            ]# cd /mnt/
                            mnt]# ls
                            lost+found

        ```

