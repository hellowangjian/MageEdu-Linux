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
    + 内核：kernel (飞模块方式)
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

