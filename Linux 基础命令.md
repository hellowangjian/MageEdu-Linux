## Linux 基础命令

	第 02 天 Linux 基础命令（04）

### 基础命令
---
- `date`:
```
NAME
        date - print or set the system date and time

SYNOPSIS
        date [OPTION]... [+FORMAT]：显示
            formate：格式符号
                %D:
                    # date +%D
                    04/15/18
                %F:
                    # date +%F
                    2018-04-15
                %T:
                    # date +%T
                    19:58:47
                %H-%M-%S:
                    # date +%H-%M-%S
                    20-01-14
                %F-%H-%M-%S:
                    # date +%F-%H-%M-%S
                    2018-04-15-20-01-45
        date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]：设置
            MM：月份
            DD：日期
            hh：小时
            mm：分钟
            yy：两位年份
            CCYY：四位年份
            .ss：秒钟
```

- Linux的两种时钟：
    + 系统时钟：有Linux内核通过CPU的工作频率进行的计时；
    + 硬件时钟：BIOS 计时器，嵌在主板上的特殊的电路；
        * `hwclock`：显示硬件时钟：
            - `-s, --hctosys`
            - `-w, --systohc`
    + `cal`：日历
    ```
        # cal
             April 2018     
        Su Mo Tu We Th Fr Sa
         1  2  3  4  5  6  7
         8  9 10 11 12 13 14
        15 16 17 18 19 20 21
        22 23 24 25 26 27 28
        29 30

        # cal 2018
                                       2018                               

               January               February                 March       
        Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
            1  2  3  4  5  6                1  2  3                1  2  3
         7  8  9 10 11 12 13    4  5  6  7  8  9 10    4  5  6  7  8  9 10
        14 15 16 17 18 19 20   11 12 13 14 15 16 17   11 12 13 14 15 16 17
        21 22 23 24 25 26 27   18 19 20 21 22 23 24   18 19 20 21 22 23 24
        28 29 30 31            25 26 27 28            25 26 27 28 29 30 31

                April                   May                   June        
        Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
         1  2  3  4  5  6  7          1  2  3  4  5                   1  2
         8  9 10 11 12 13 14    6  7  8  9 10 11 12    3  4  5  6  7  8  9
        15 16 17 18 19 20 21   13 14 15 16 17 18 19   10 11 12 13 14 15 16
        22 23 24 25 26 27 28   20 21 22 23 24 25 26   17 18 19 20 21 22 23
        29 30                  27 28 29 30 31         24 25 26 27 28 29 30

                July                  August                September     
        Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
         1  2  3  4  5  6  7             1  2  3  4                      1
         8  9 10 11 12 13 14    5  6  7  8  9 10 11    2  3  4  5  6  7  8
        15 16 17 18 19 20 21   12 13 14 15 16 17 18    9 10 11 12 13 14 15
        22 23 24 25 26 27 28   19 20 21 22 23 24 25   16 17 18 19 20 21 22
        29 30 31               26 27 28 29 30 31      23 24 25 26 27 28 29
                                                      30
               October               November               December      
        Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
            1  2  3  4  5  6                1  2  3                      1
         7  8  9 10 11 12 13    4  5  6  7  8  9 10    2  3  4  5  6  7  8
        14 15 16 17 18 19 20   11 12 13 14 15 16 17    9 10 11 12 13 14 15
        21 22 23 24 25 26 27   18 19 20 21 22 23 24   16 17 18 19 20 21 22
        28 29 30 31            25 26 27 28 29 30      23 24 25 26 27 28 29
                                                      30 31
    ```

- 目录相关的命令：
    + 当前目录或工作目录
    + 主目录，家目录: HOME
        * root：/root
        * 普通用户：/home/USERNAME
            - `/home/tom`
        * `~`：用户的主目录
    + `cd`
        * `cd` 或 `cd ~`：切换至当前用户的主目录；
        * `cd ~USERNAME`：切换至指定用户的主目录（仅限 root 用户使用）；
        * `cd -`：在上一个目录与当前目录之间来回切换；
        * `.`：当前目录；
        * `..`：上一级目录；
        * 相关的环境变量：
            - `PWD`：保存了当前目录路径；
            - `OLDPWD`：上一次所在的工作目录路径；
            ```
                [root@Testserver ~]# cd /var/
                [root@Testserver var]# echo $PWD
                /var
                [root@Testserver var]# echo $OLDPWD
                /root
            ```
    + `pwd`：显示当前目录；
    ```
        [root@Testserver var]# pwd
        /var
    ```
    + `ls`：list，显示指定路径下的文件列表；
    ```
        ls [OPTION]... [DIR]...
            -a, --all：显示所有文件，包括以点(.)开头的隐藏文件；
            -l：长格式；
                -rw-r--r-- 1 root    root        8785 Sep  3  2015 install.log
                    -rw-r--r--：
                        最左侧的第一位：文件类型
                            -, d, l, b, c, p, s
                        后面的9位：访问权限，perm
                数字：文件被硬链接的次数；
                root：文件的owner；
                root：文件的group；
                8785：文件的size；
                Sep  3  2015：文件的最近一次被修改的时间；
            -h, --human-readable：单位换算；
            -d：显示目录自身的相关属性，通常要与 -l 一起使用；
            -r, --reverse：逆序显示；
            -R, --recursive：递归显示；
    ```
    + `stat`：获取指定文件的元数据信息；
    ```
        stat [OPTION]... FILE...

        ~]# stat install.log
          File: `install.log'
          Size: 8785            Blocks: 24         IO Block: 4096   regular file
        Device: 802h/2050d      Inode: 2293762     Links: 1
        Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
        Access: 2017-09-30 09:44:30.000000000 +0800
        Modify: 2015-09-03 05:59:36.000000000 +0800
        Change: 2015-09-03 05:59:39.000000000 +0800
    ```

- 文件查看的命令
    + `cat`：concatenate files and print on the standard output
    ```
        cat [OPTION]... [FILE]...
            -E：显示行结束符"$"；
                # cat -E /etc/issue
                CentOS release 6.7 (Final)$
                Kernel \r on an \m$
                $
            -n：对显示出的每一行进行编号；
                # cat -n /etc/issue
                 1  CentOS release 6.7 (Final)
                 2  Kernel \r on an \m
                 3
    ```
    + tac：concatenate and print files in reverse
    ```
        # tac /etc/issue

        Kernel \r on an \m
        CentOS release 6.7 (Final)
    ```

- 文件内容类型查看命令：
    + `file`：determine file type
    ```
        # ls -ld /etc/issue /etc /dev/sda /bin/cat
        -rwxr-xr-x.   1 root root 45224 Oct 15  2014 /bin/cat
        brw-rw----    1 root disk  8, 0 Apr 11 10:09 /dev/sda
        drwxr-xr-x. 101 root root 12288 Apr  8 13:29 /etc
        -rw-r--r--.   1 root root    47 Aug  4  2015 /etc/issue

        [root@Testserver ~]# file /etc/issue
        /etc/issue: ASCII text
        [root@Testserver ~]# file /etc/
        /etc/: directory
        [root@Testserver ~]# file /dev/sda
        /dev/sda: block special
        [root@Testserver ~]# file /bin/cat
        /bin/cat: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, stripped
    ```

- 回显命令：
    + `echo`
    ```
        [root@Testserver ~]# echo "how are you?"
        how are you?

        -n：禁止自动添加换行符号；
            [root@Testserver ~]# echo -n "how are you?"
            how are you?[root@Testserver ~]# 
        -e：允许使用转义符；
            \n：换行；
            \t：制表符；
            [root@Testserver ~]# echo "how \n are you"
            how \n are you
            [root@Testserver ~]# echo -e "how \n are you" 
            how 
             are you

        echo "$VAR_NAME"：变量会替换，双引号表示弱引用；
        echo '$VAR_NAME'：变量不会替换，强引用；
    ```

- 查找命令的命令
    + `which`：显示命令对应的程序文件路径；
    ```
        which [options] [--] programname [...]

            ~]# which ls
            alias ls='ls --color=auto'
                    /bin/ls

            --skip-alias：不予显示别名；
                ~]# which --skip-alias ls
                /bin/ls
    ```
    + `whereis`：locate the binary, source, and manual page files for a command；
    ```
        whereis [-bmsu] [-BMS directory...  -f] filename...

             ~]# whereis ls
            ls: /bin/ls /usr/share/man/man1p/ls.1p.gz /usr/share/man/man1/ls.1.gz

            -b：Search only for binaries.
                ~]# whereis -b ls
                ls: /bin/ls
            -m：Search only for manual sections.
                ~]# whereis -m ls
                ls: /usr/share/man/man1p/ls.1p.gz /usr/share/man/man1/ls.1.gz
    ```
    + `whatis`：search the whatis database for complete words.
    ```
        使用 makewhatis 命令可将当前系统上所有的帮助手册及与之对应的关键字创建为一个数据库；

        ~]# whatis read
        read                 (1p)  - read a line from standard input
        read                 (2)  - read from a file descriptor
        read                 (3p)  - read from a file
        read [builtins]      (1)  - bash built-in commands, see bash(1)
    ```

```
第 03 天 【linux文件系统及文件类型(01)】
```

- 系统管理类命令：
    + 关机：
        * halt：
            - `-f`：强制，不调用 shutdown 命令；
            - `-p`：切断电源；
        * poweroff
        * shutdown：
        ```
            shutdown [OPTION]...  TIME [MESSAGE]

            -r：reboot
            -h：halt
            -c：cancel

            TIME：
                now：立刻
                +m：相对时间表示法，从命令提交开始多久之后；例如 +3；
                hh:mm：绝对时间表示，指明具体时间；
        ```
        * init 0
    + 重启：
        * reboot
        * shutdown
        * init 6
    + 跟用户登录相关：
        * who：系统当前所有的登录会话；
        ```
            # who
            root     tty1         2017-11-01 15:49
            root     pts/0        2018-05-20 13:47 (114.89.64.142)
        ```
        * whoami：显示当前登录的有效用户；
        ```
            # whoami
            root
        ```
        * w：系统当前所有的登录会话及所做的操作；
        ```
            # w
             14:34:59 up 669 days, 20:16,  2 users,  load average: 0.08, 0.06, 0.01
            USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
            root     tty1     -                01Nov17 199days  0.01s  0.01s -bash
            root     pts/0    114.89.64.142    13:47    0.00s  0.04s  0.00s w
        ```

