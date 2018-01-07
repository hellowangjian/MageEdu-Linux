## Linux 命令帮助获取

    第 02 天 【Linux命令帮助的获取详解(03)】

### Linux 命令帮助的获取

- 格式：COMMAND [OPTIONS...] [ARGUMENTS..]
    + 内部命令：
        * `# help COMMAND`
            - hash 命令：Remember or display program locations.
                + shell 搜寻到的外部命令结果会缓存至 kv(key-velue) 存储中；
                ```
                # help hash
                    Remember or display program locations.
                    -d                forget the remembered location of each NAME 
                    -r                forget all remembered locations
                # hash -d ls  // 从缓存中删除 ls 命令的缓存
                # hash -r     // 从缓存中删除所有命令的缓存
                ```
    + 外部命令：都由一个可执行程序，位于文件系统某目录下；
        * 可以使用`which`,`whereis`命令查找； 
        * shell 程序搜寻可执行程序文件的路径定义在 PATH 环境变量中；
            - `# echo $PATH`
            - 注意：自左至右查找
        * （1）`# COMMAND --help; # COMMAND -h`
        * （2）使用手册（manual）
            - `# man COMMAND`
        * （3）信息页
            - `# info COMMAND`
        * （4）程序自身的帮助文档
            - README
            - INSTALL
            - ChangeLog
        * （5）程序官方文档
            - 官方站点：Documentation
        * （6）发行版的官方文档
        * （7）Google

- 