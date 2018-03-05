## Docker

    docker(1) 【docker 基础原理（01）】

- Namespace: 内核级别，环境隔离；
    + PID NameSpace: Linux 2.6.24 , PID隔离；
    + Network NameSpace: Linux 2.6.29 , 网络设备、网络栈、端口等网络资源隔离；
    + User NameSpace: Linux 3.8 , 用户和用户组资源隔离；
    + IPC NameSpace: Linux 2.6.19 , 信号量、消息队列和共享内存的隔离；
    + UTS NameSpace: Linux 2.16.19 , 主机名和域名的隔离；
    + Mount NameSpace: Linux 2.4.19 , 挂载点（文件系统）隔离；

    + API: clone(), setns(), unshare()

- CGroup: Linux Control Group, 控制组, Linux 2.6.24
    + 内核级别，限制，控制与分离一个进程族群的资源；

    + 资源：CPU, 内存, IO
    + 功能：
        * Resource limitation：资源限制；
        * Prioritization：优先级控制；
        * Accounting：审计和统计，主要为计费；
        * Control：挂起进程，恢复进程；
    + /sys/fs/cgroup
    + mount
    + lssubsys -m
    + CGroup 的子系统（subsystem）：
        * blkio：设定块设备的 IO 限制；
        * cpu：设定 CPU 的限制；
        * cpuacct：报告 cgroup 中所使用的 CPU 资源；
        * cpuset：为 cgroup 中的任务分配 CPU 和内存资源；
        * memory：设定内存的使用限制；
        * devices：控制cgroup中的任务对设备的访问；
        * freezer：挂起或恢复cgroup中的任务；
        * net_cls：（classid），使用等级级别标识符来标记网络数据包，以实现基于 tc 完成对 cgroup 中产生的流量的控制；
        * perf_event：使用后使 cgroup 总的任务可以进行统一的性能测试；
        * hugetlb：（Translation Look-aside Buffers）对 HugeTLB 系统进行限制；
    + CGroup 中的术语：
        * task（任务）：进程或线程；
        * cgroup：一个独立的资源控制单位，可以包含一个或多个子系统；
        * subsystem：子系统；
        * hierarchy：层级；

- AUFS：UnionFS
    + UnionFS：把不同的物理位置的目录合并到同一个目录中。
    + Another UFS, Alternative UFS, Advanced UFS

- Device mapper: Linux 2.6内核引入的最重要的技术之一，用于在内核中支持逻辑卷管理的通用设备映射机制；
    + Mapped Devide
    + Mapping Table
    + Target Device

### Docker:
    2013年，GO 语言开发，遵循 Apache 2.0 开源协议，由 dotCloud 开发。

- C/S架构：
    + Docker Client: 发起docker相关的请求；
    + Docker Server: 容器运行的节点；


>    docker(1) 【docker 使用入门（02）】    

- namespace, cgroup
    + 解决方案：
        * lxc, openvz
        * lxc: linux containers
        * libcontainer

- 核心组件：
    + docker client：docker的客户端工具，是用户使用docker的主要接口，docker client 与 docker daemon 通信并将结果返回给用户；
    + docker deamon：运行于宿主机上，Docker守护进程，用户可通过docker client 与其交互；
    + image：镜像文件是只读的；用来创建container，一个镜像可以运行多个container；镜像文件可以通过 Dockerfile 文件创建，也可以从 docker hub/registry 下载；
    + repository
        * 公共仓库：Docker hub/registry
        * 私有仓库：docker registry

    + docker container：docker的运行实例，容器是一个隔离环境；
    + 另外两个重要组件：
        * docker link：
        * docker volume：

    


