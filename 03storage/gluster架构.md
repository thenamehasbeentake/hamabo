##　架构

### 特点
分布式文件存储
开源，横向拓展，支持多客户端通过无线带宽RDMA或者TCP/IP方式将廉价主机以分布式的方式连接起来


[官方文档架构介绍](https://docs.gluster.org/en/latest/Quick-Start-Guide/Architecture/)

### 整体：
~~~
用户程序 ->　client -> server

用户态用户程序，系统调用操作glusterfs客户端挂载的目录

调用客户端 内核态vfs，vfs再调用内核的fuse模块，fuse模块将操作作用在/dev/fuse设备文件上，而我们的用户态glusterfs客户端进程poll轮询监听/dev/fuse设备，响应操作。

用户态glusterfs客户端通过tcp或者rdma将数据发给服务端，服务端通过系统调用内核vfs接口，在到内核底层支持的原始文件系统
~~~

xlator架构
![xlator](https://docs.gluster.org/en/latest/images/GlusterFS_Translator_Stack.png)

client

fuse bridge

io-stats    调试用的

performance/cache xlator
md-cache    元数据缓存matedata
open-behind     延迟open，除非遇到writev,unlink这样必须open的才会open
io-cache    数据缓存
read-ahead  预读
readdir-ahead   预读目录
write-behind  延迟fflush

shrad切片，大文件切成很多小文件
dht　   一致性哈希
afr 副本
client 网络通信


server：
quota 配额
index
barrier
marker 异地备份
selinux
iothreads 多线程
upcall
leases
read-only
worm    删除限制
locks   协同锁和强自锁
access-control
bitrot  静默损坏
changelog
trashcan    垃圾回收

posix



## 优缺点
- 用户态文件系统fuse
    安装使用简单，对开发者友好。
    对性能会有影响
- 堆栈式结构
    模块化堆栈式架构，每个xlator是一个动态库，配置该xlator后可以在配置文件中看到对应信息。通过特定的拓扑结构组织一组xlator，构建出glusterfs卷。应对不同场景灵活性较高。
    优点：系统拓展能力强，系统设计实现复杂性降低
    缺点：效率较低，xlator树深度通常10层以上，一层层调用，效率较差
- 无元数据节点
    元数据存在于文件的属性和扩展属性中。当需要访问某文件时，客户端使用DHT算法，根据文件的路径和文件名计算出文件所在brick，然后由客户端从此brick获取数据，省去了同元数据服务器通信的过程。
    优点：无元数据服务器瓶颈。采用无中心对称式架构，没有专用的元数据服务器，也就不存在元数据服务器瓶颈。
    缺点：ls操作会遍历整个集群

- 弹性hash算法代替传统的有元数据节点服务
    优点：获得了接近线性的高扩展性
    缺点：拓展之后新节点负载较重。

- 高可用
    支持副本、EC等冗余设计。保证在冗余范围内的节点掉线时，仍然可以从其它服务节点获取数据，保证高可用性。采用弱一致性的设计，当向副本中文件写入数据时，客户端计算出文件所在brick，然后通过网络把数据传给所在brick，当其中有一个成功返回，就认为数据成功写入，不必等待其它brick返回，就会避免当某个节点网络异常或磁盘损坏时因为一个brick没有成功写入而导致写操作等待。
    服务器端还会随着存储池的启动，而开启一个glustershd进程，这个进程会定期检查副本和EC卷中各个brick之间数据的一致性，并恢复。
- 存储池功能丰富，完整的存储操作系统栈
    基本的dht，可以和ec、afr自由组合，并提供shard切片功能
- 高性能
    采用弱一致性的设计，向副本中写数据时，只要有一个brick成功返回，就认为写入成功，不必等待其它brick返回，这样的方式比强一致性要快。
还提供了I/O并发、write-behind、read-ahead、io-cache、条带等提高读写性能的技术。并且这些都还可以根据实际需求进行开启/关闭，i/o并发数量，cache大小都可以调整。
- 原始数据格式存储（DataStored in Native Formats）
    底层brick对应原始数据格式存储

- 官方文档上写的优点：
Gluster is a scalable, distributed file system that aggregates disk storage resources from multiple servers into a single global namespace.
Gluster 是一个可扩展的分布式文件系统，它将来自多个服务器的磁盘存储资源聚合到一个全局命名空间中。
- Advantages
    * Scales to several petabytes   扩展到数 PB 
    * Handles thousands of clients  处理数以千计的客户端
    * POSIX compatible      POSIX 兼容 
    * Uses commodity hardware   在买到的硬件上都能跑
    * Can use any ondisk filesystem that supports extended attributes 可以使用任何支持扩展属性的磁盘文件系统 
    * Accessible using industry standard protocols like NFS and SMB  可使用 NFS 和 SMB 等行业标准协议访问 
    * Provides replication, quotas, geo-replication, snapshots and bitrot detection 提供复制、配额、异地复制、快照和静默损坏
    * Allows optimization for different workloads   允许针对不同的工作负载进行优化 
    * Open Source   开源

### 缺点：
- 扩容、缩容时影响的服务器较多
Glusterfs对逻辑卷中的存储单元brick划分hash分布空间（会以brick所在磁盘大小作为权重，空间总范围为0至232-1），一个brick占一份空间，当访问某文件时，使用Davies-Meyer算法根据文件名计算出hash值，比较hash值落在哪个范围内，即可确定文件所在的brick，这样定位文件会很快。但是在向逻辑卷中添加或移除brick时，hash分布空间会重新计算，每个brick的hash范围都会变化，文件定位就会失败，因此需要遍历文件，把文件移动到正确的hash分布范围对应的brick上，移动的文件可能会很多，加重系统负载，影响到正常的文件访问操作。

- 遍历目录下文件耗时
1.Glusterfs没有元数据节点，而是根据hash算法来确定文件的分布，目录利用扩展属性记录子卷的中brick的hash分布范围，每个brick的范围均不重叠。遍历目录时，需要readdir子卷中每个brick中的目录，获取每个文件的属性和扩展属性，然后进行聚合，相对于有专门元数据节点的分布式存储，遍历效率会差很多，当目录下有大量文件时，遍历会非常缓慢。
2.删除目录也会遇到同样的问题。
3.目前提供的解决方法是合理组织目录结构，目录层级不要太深，目录下文件数量不要太多，增大glusterfs目录缓存。另外，还可以设计把元数据和数据分离，把元数据放到内存数据库中（如redis、memcache）,并在ssd上持久保存。

- 小文件性能较差
    1. Glusterfs主要是为大文件设计，如io-cache、read-ahead、write-behind和条带等都是为优化大文件访问，对小文件的优化很少。
    2. Glusterfs采用无元数据节点的设计，文件的元数据和数据一起保存在文件中，访问元数据和数据的速率相同，访问元数据的时间与访问数据的时间比例会较大，而有元数据中心的分布式存储系统，对元数据服务器可以采用更快速的ssd盘，可以采用更大的元数据缓存等优化措施来减小访问元数据时间与访问数据时间的比值，来提高小文件性能。
    3. Glusterfs在客户端采用了元数据缓存md-cache来提高小文件性能，但是md-cache大小有限，但在海量小文件场景下，缓存命中率会严重下降，优化效果会减小，这就需要增大元数据缓存。
    4. 针对小文件性能差的问题，也可以参考TFS的做法， TFS会将大量的小文件合并成一个大文件，这个大文件称为Block，每个Block拥有在集群内唯一的编号（Block Id），Block Id在NameServer创建Block时分配，NameServer维护block与DataServer（存储Block的实际数据）的关系。



[参考1](https://www.jianshu.com/p/a33ff57f32df)

[参考2](https://blog.51cto.com/dangzhiqiang/1907174)


