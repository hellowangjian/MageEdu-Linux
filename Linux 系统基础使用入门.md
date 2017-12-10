Linux 系统基础使用入门

    第 02 天 【Linux系统基础使用入门(02)】

- 终端：用户与主机交互，必然用到的设备；
    + 物理终端：直接接入本机的显示器和键盘设备；
        * 设备文件路径：`/dev/console`
    + 虚拟终端：附加在物理终端之上的以软件方式虚拟实现的终端，CentOS 6 默认启动 6 个虚拟终端。  
        `切换：Ctrl + Alt + F#: [1,6]`
        * 设备文件路径：`/dev/tty#: [1,6]`
    + 图形终端：附加在物理终端之上的以软件方式虚拟实现的终端，但额外会提供桌面环境；  
        `切换：Ctrl + Alt + F7`
    + 模拟终端：
        * 图形界面下打开的命令行接口，基于 ssh 协议或 telnet 协议等远程打开的界面；
        * 设备文件路径：`/dev/pts/#: [0,oo]`
    + 查看当前使用的终端类型命令：`tty`

- 交互式接口：启动终端后，在终端设备附加一个交互式应用程序
    + GUI：
        * X protocol, window manager, desktop
        * Disktop：
            - GNOME (c,图形开发库：gtk)
            - KDE   (c++, 图形开发库：qt)
            - XFCE  (轻量级桌面)
    + CLI：
        * shell 程序：
            - sh (bourn)
            - csh
            - tcsh
            - ksh (korn)
            - bash (bourn again shell), GPL
            - zsh
        * 显示当前使用的 shell：
             
        ```
            # echo $SHELL      
            /bin/bash    
        ```

        * 显示当前系统使用的所有 shell：  
        
        ```
            # cat /etc/shells   
            /bin/sh  
            /bin/bash  
            /sbin/nologin  
            /bin/dash       
            /bin/false    
        ```

        * 命令提示符


