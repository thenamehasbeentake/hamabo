# 存储，从入门到精通
## 操作系统有关

### 文件系统和posix接口，相关linux系统调用
常见posix接口？什么是文件系统？文件系统四大组成？常见文件系统，文件系统优缺点？什么是系统调用？lnux存储堆栈图。
[posix接口](https://book.douban.com/subject/25900403/)
[文件系统](https://doc.lagout.org/operating%20system%20/linux/Understanding%20Linux%20Kernel.pdf)
[常见文件系统](https://elixir.bootlin.com/linux/latest/source/fs)
[linux存储堆栈图](https://www.thomas-krenn.com/en/wiki/Linux_Storage_Stack_Diagram)




## 工具
### 性能监测工具
#### 压测工具
- vdbench、iozone、dd、fio、cosbench
关注点：文件大小、io块大小、多少并发、多少层目录、顺序随机覆盖等等、读写等等
#### 监控工具
- blktrace、perf、iperf、top一系列、dstat、vmstat、iostat
关注点：缓存、网络、磁盘、内存、cpu负载，有没有读写放大问题









## glusterfs入门
### glusterfs安装、功能、进程关系
代码的编译安装，集群的搭建，各个进程的关系
[编译安装](https://docs.gluster.org/en/latest/Developer-guide/Building-GlusterFS/)
[进程关系](https://docs.gluster.org/en/latest/Quick-Start-Guide/Architecture/)
[集群管理搭建](https://docs.gluster.org/en/latest/Administrator-Guide/)
[打包文档](http://20.20.20.2:4999/web/#/7?page_id=96)

### 硬件arm架构，a72，385芯片
不需要过度深入，主要是在不同硬件下代码的适配打包工作，可能会有一些函数或者宏需要调整

## 第二阶段，搭建集群，会用glusterfs
### gluster集群搭建
### gluster命令
### 看官方文档
### 常见问题的处理
[副本卷脑裂](http://20.20.20.2:4999/web/#/7?page_id=329)
卷配置文件不一样
扩容问题
数据损坏：操作系统盘损坏、数据损坏、brick配置文件损坏

## 第三阶段，学习原理，阅读代码
要先了解清楚原理，看代码才能事半功倍。
可以看官方文档 开发者导引
### fuse
### glusterfs架构
堆栈式xlator怎么实现的。glusterd进程
### dht
一致性哈希
### ec，afr
ec：纠删码编码解码
afr:副本卷仲裁，脑裂
### posix
底层对接、aio、io:uring

### 性能调优，定位瓶颈
gluster v profile命令
blktrace命令

### 拓宽视野
了解其他厂商存储产品，了解存储有关硬件、存储软件常见技术、分布式技术、前沿的技术。开源分布式文件对象块存储软件区别，优缺点。
