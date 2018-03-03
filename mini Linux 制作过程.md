## mini Linux制作过程

    第 25 天 mini Linux制作过程（01）

### Mini Linux

- 启动流程：
    + CentOS 6:
    ```
        POST --> BostSequence(基于 BIOS 设定) --> BootLoader --> Kernel（ramdisk）--> rootfs --> /sbin/init

            编写服务脚本
            upstart配置文件

            /etc/init/*.conf
            /etc/inittab
                默认运行级别
                运行系统初始化脚本：/etc/rc.d/rc.sysinit
                /etc/rc.d/rc $runlevel
                启动终端，并运行login
                启动图形终端

        bootloader:
            lilo
            grub legacy
            grub2：
                stage1: mbr
                stage1_5: filesystem driver
                stage2: grub main program
    ```
    + CentOS 7:
    ```
        POST --> BostSequence(基于 BIOS 设定) --> BootLoader --> Kernel（ramdisk）--> rootfs --> /sbin/systemd

            编写服务脚本
            systemd unit 文件
    ```

- 内核编译：
    + make menuconfig --> .config
    + make [ -j # ]
    + make modules_install
    + make install

- 制作 mini Linux 步骤：
    + bootloader：grub
    + 内核：kernel (非模块方式)
    + 根文件系统：busybox

### 编译内核

- 安装开发环境：
```
    ~]# yum groupinstall "Development Tools" "Server Platform Development" -y
```

- 下载、解压、建立软连接内核源码
```
    1、下载
        ~]# wget http://140.206.33.170:8081/pub/kernel/linux-3.10.105.tar.xz
        or
        去内核官网(https://www.kernel.org)下载。
    2、解压至 /usr/src 目录下
         ~]# tar xf linux-3.10.105.tar.xz -C /usr/src
    3、建立连接
        ~]# cd /usr/src/
        src]# ln -sv linux-3.10.105  linux
        `linux' -> `linux-3.10.105'
```

- 查看硬件设备类型
```
    为了使内核轻量，我们只将需要的硬件设备驱动编译进内核，或者启动对相关硬件设备的支持。
    可以使用 lscpu、lspci 等命令获取当前系统的硬件信息。
```

- 编译内核
    + 创建内核配置模版文件
        * 将所有可选的配置全部禁用
        ```
        src]# cd linux
        linux]# make allnoconfig   
        ```
        * 图形化配置可选项
        ```
        linux]# make menuconfig    
        ```
        ![menuconfig](images/menuconfig.png)
            - 编译为 64 位内核
            ![menuconfig_kernel](images/menuconfig_kernel.png)
            - 启用加载模块支持
            ![menuconfig_enable_loadable_module_support.png](images/menuconfig_enable_loadable_module_support.png)
                + 支持模块动态装卸载
                ![menuconfig_module_unloading.png](images/menuconfig_module_unloading.png)
            - Processor type and features > Symmetric multi-processing support（使内核支持多核cpu功能）
            ![menuconfig_symmetric_multi-processing_support.png](images/menuconfig_symmetric_multi-processing_support.png)
            - Bus options (PCI etc.) > PCI support（开启支持PCI）
            ![menuconfig_symmetric_pci_support.png](images/menuconfig_symmetric_pci_support.png)
            - Device Drivers > SCSI device support（开启支持SCSI设备）
            ![menuconfig_scsi_device_support.png](images/menuconfig_scsi_device_support.png)
            - Device Drivers > SCSI device support > SCSI disk support（开启SCSI磁盘的支持）
            ![menuconfig_scsi_disk_support.png](images/menuconfig_scsi_disk_support.png)
            - Device Drivers > Fusion MPT device support > 一系列Fusion MPT驱动全部选择
            ![menuconfig_fusion_mpt_device_support.png](images/menuconfig_fusion_mpt_device_support.png)
    + 编译
    ```
        linux] make -j 4 bzImage //编译为bz2压缩格式的image，默认存储为 arch/x86/boot/bzImage
    ```

- 给当前的虚拟机（CentOS 6）添加一块新的硬盘，此硬盘作为 Mini Linux 内核的安装
![menuconfig_minilinux_vmdk.png](images/menuconfig_minilinux_vmdk.png)
    + 分区：
    ```
        [root@node1 ~]# fdisk /dev/sdb 
        Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
        Building a new DOS disklabel with disk identifier 0x9e1fcfba.
        Changes will remain in memory only, until you decide to write them.
        After that, of course, the previous content won't be recoverable.

        Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

        WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
                 switch off the mode (command 'c') and change display units to
                 sectors (command 'u').

        Command (m for help): n
        Command action
           e   extended
           p   primary partition (1-4)
        p
        Partition number (1-4): 1
        First cylinder (1-2610, default 1): 
        Using default value 1
        Last cylinder, +cylinders or +size{K,M,G} (1-2610, default 2610): +100M

        Command (m for help): n
        Command action
           e   extended
           p   primary partition (1-4)
        p
        Partition number (1-4): 2
        First cylinder (15-2610, default 15): 
        Using default value 15
        Last cylinder, +cylinders or +size{K,M,G} (15-2610, default 2610): +2G

        Command (m for help): p

        Disk /dev/sdb: 21.5 GB, 21474836480 bytes
        255 heads, 63 sectors/track, 2610 cylinders
        Units = cylinders of 16065 * 512 = 8225280 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disk identifier: 0x9e1fcfba

           Device Boot      Start         End      Blocks   Id  System
        /dev/sdb1               1          14      112423+  83  Linux
        /dev/sdb2              15         276     2104515   83  Linux

        Command (m for help): w
        The partition table has been altered!

        Calling ioctl() to re-read partition table.
        Syncing disks.
    ```
    + 格式化：
    ```
        ~]# mke2fs -t ext4 /dev/sdb1
        ~]# mke2fs -t ext4 /dev/sdb2
    ```
    + 挂载：
    ```
        ~]# mkdir /mnt/{boot,sysroot}
        ~]# mount /dev/sdb1 /mnt/boot/    // 此作为 Mini Linux 的 boot 分区
        ~]# mount /dev/sdb2 /mnt/sysroot/
    ```

- 安装`grub`到`/mnt/boot/`目录:
```
    ~]# grub-install --root-directory=/mnt /dev/sdb
```

- 复制内核到 boot 目录：
```
    ~]# cp  /usr/src/linux/arch/x86/boot/bzImage /mnt/boot/bzImage
    ~]# file /mnt/boot/bzImage
    /mnt/boot/bzImage: Linux kernel x86 boot executable bzImage, version 3.10.105 (root@node1.centos.cn), RO-rootFS, swap_dev 0x1, Normal VGA
```

- 配置`grub.conf`文件：
```
    ~]# vim /mnt/boot/grub/grub.conf
    default=0
    timeout=3
    title Mini Linux (3.10.105)
            root (hd0,0)
            kernel /bzImage ro root=/dev/sda2
```

至此编译内核已经完成，并新增加了一块硬盘，新的硬盘作为 Mini Linux 的系统安装盘，为其创建了 `/boot` 和 `/` 两个分区，然后为 `/boot` 分区安装了 `grub`，最后将新编译的内核镜像文件复制至 `/boot` 分区，并为 `grub` 编写了 `grub.conf` 配置文件，使其能够使用新编译的内核镜像。

下面将 node1 **挂起**，并创建 Mini Linux 虚拟机，并使用上述配置的硬盘作为系统盘。

启动 Mini Linux，`grub` 已能正常运行，并提示使用的内核文件：

![start_minilinux.png](images/start_minilinux.png)

因编译内核是没有启用对文件系统的驱动，所以这里会显示根文件系统无法挂载，内核恐慌（kernel panic）：
![kernel_panic.png](images/kernel_panic.png)

- 重新编译内核，将文件系统驱动增加进去：

    将 Mini Linux 关闭，开启 node1 后，重新编译内核。
    ```
        ~]# cd /usr/src/linux
        linux]# make menuconfig
    ```

    + File systemx > 需要支持的文件系统
    ![menuconfig_file_system.png](images/menuconfig_file_system.png)

    + Executable file formats / Emulations > 开启内核可执行文件的格式（ELF 和以 #! 开头的 shell 脚本）
    ![menuconfig_executable_file_formats.png](images/menuconfig_executable_file_formats.png)

- 重新编译，并将 `bzImage` 文件复制至 `/mnt/boot/` 目录
```
    linux]# make -j 4 bzImage 
    ~]# cp arch/x86/boot/bzImage /mnt/boot/
    cp: overwrite `/mnt/boot/bzImage'? y
```

- 此时，将 node1 挂起，再次启动 Mini Linux，其因为没有 `init` 程序，导致内核再次恐慌：
![kernel_panic2.png](images/kernel_panic2.png)

- 源码中定义的内核启动后执行的命令：
```
    ~]# vim /usr/src/linux/init/main.c
    841         if (execute_command) {
    842                 if (!run_init_process(execute_command))
    843                         return 0;
    844                 pr_err("Failed to execute %s.  Attempting defaults...\n",
    845                         execute_command);
    846         }
    847         if (!run_init_process("/sbin/init") ||
    848             !run_init_process("/etc/init") ||
    849             !run_init_process("/bin/init") ||
    850             !run_init_process("/bin/sh"))
    851                 return 0;
```

- 移植一个 `bash` 至 Mini Linux：
    + 手动创建一个跟文件系统：
    ```
        ~]# cd /mnt/sysroot/
        sysroot]# mkdir -pv etc dev proc sys bin sbin usr/{bin,sbin,lib,lib64} lib64 lib/modules home var/{log,run,lock} tmp mnt media root
    ```
    + 复制当前系统的 `bash`
    ```
    复制程序及其依赖的库文件脚本示例：
    ~]# cat bincp.sh 
    #!/bin/bash
    #
    target=/mnt/sysroot
    [ -d $target ] || mkdir $target

    read -p "A command: " command

    libcp() {
            for lib in $(ldd $1 | grep -o "[^[:space:]]*/lib[^[:space:]]*"); do
                    libdir=$(dirname $lib)
                    [ -d $target$libdir ] || mkdir -p $target$libdir
                    [ -f $target$lib ] || cp $lib $target$lib
            done
    }

    while [ "$command" != 'quit' ]; do
            if ! which $command &> /dev/null; then
                    read -p "No such command, enter again:" command
                    continue
            fi
            command=$(which --skip-alias $command)
            cmnddir=$(dirname $command)

            [ -d $cmnddir ] || mkdir -p $target$cmnddir
            [ -f $target$command ] || cp $command $target$command
            libcp $command
            read -p "Another command(quit):" command
    done

    ~]# bash bincp.sh 
    A command: bash
    Another command(quit):ls
    Another command(quit):quit
    ~]#
    ```

- 修改 `grub.conf` 文件，指定其 `init` 运行 `/bin/bash`：
```
    ~]# vim /mnt/boot/grub/grub.conf
    default=0
    timeout=3
    title Mini Linux (3.10.105)
            root (hd0,0)
            kernel /bzImage ro root=/dev/sda2 init=/bin/bash
```

- 将 node1 挂起，再次启动 Mini Linux，显示已经启动 `bash`：
![start_minilinux_bash.png](images/start_minilinux_bash.png)

此时还无法对 Mini Linux 输入任何内容，因为编译内核时，没有启用键盘驱动。

- 重新编译内核，加入键盘及鼠标的驱动：
![menuconfig_keyboards.png](images/menuconfig_keyboards.png)
Ps: VMware 模拟的鼠标是 usb 接口的鼠标，如若使内核支持鼠标，需先在 `Device Driver` 中启用 usb 驱动，可以使用 `lsusb` 命令查看虚拟机使用 usb 类型。

- 再次复制内核镜像文件至 `/mnt/boot` 目录后，挂起 node1，启动 Mini Linux：
![start_minilinux_bash2.png](images/start_minilinux_bash2.png)

此时，内核成功启动完成。

- 给 Mini Linux 仿造一个 init 脚本文件，以供其启动时自动启动：
```
    ~]# cd /mnt/sysroot/
    sysroot]# vim sbin/init
    #!/bin/bash
    #
    echo -e "\tWelcome to \033[32mMini\033[0m Linux"
    mount -n -t proc proc /proc
    mount -n -t sysfs sysfs /sys
    mount -n -o remount,rw /dev/sda2 /
    /bin/bash
    sysroot]# chmod +x sbin/init

    删除 grub.conf 中的 init 设置，使其使用默认值会自动执行 /sbin/init 文件：
        ~]# vim /mnt/boot/grub/grub.conf
        default=0
        timeout=3
        title Mini Linux (3.10.105)
                root (hd0,0)
                kernel /bzImage ro root=/dev/sda2 

    ~]# bash bincp.sh 
    A command: mount
    Another command(quit):blkid
    Another command(quit):touch
    Another command(quit):quit
```

此时如果重新启动 Mini Linux，其 `/dev/` 目录下没有任何设备文件，需要内核支持，内核会对识别的设备自动在 `/dev/` 目录下创建设备文件。

- 内核启用支持自动在 `/dev/` 自动创建设备文件：
![menuconfig_generic_driver_options.png](images/menuconfig_generic_driver_options.png)

- 编译 --> 复制内核文件至 `/mnt/boot` --> 挂起 node1，启动 Mini Linux：
![start_minilinux_init.png](images/start_minilinux_init.png)

其执行了 `init` 文件，挂载了相应的设备，内核并在 `/dev/` 目录下输出了识别的设备的设备文件。
