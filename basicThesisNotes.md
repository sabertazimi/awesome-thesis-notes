大数据处理典型问题:

*   数据一致性问题
*   数据容错与恢复问题
*   节点通信问题
*   能耗问题

# Conference List

*   ISCA
*   HPCA
*   MICRO
*   ASPLOS
*   PPoPP
*   SC
*   ATC
*   OSDI
*   SOSP
*   FAST
*   EuroSys
*   IPDPS

# In-memory Computing Thesis Notes

内存计算特点:

*   硬件: 大内存
*   软件: 良好编程模型和编程接口
*   面向数据密集型应用: 数据规模大, 实时性要求高
*   大多支持并行处理数据

内存计算分类:

*   单节点
*   分布式集群
*   新型混合内存 (Hybrid Memory)

## 单节点内存计算

节点拥有 1/n 个处理器(单/多核) + 共享内存 (Shared Memory)

### 内存存储系统

### 内存数据处理系统

### 内存压缩技术

### 提升 I/O 访问效率

## 分布式内存计算

*   容错与恢复
*   同步与一致性
*   内存分配与管理
*   网络瓶颈

### 内存存储系统

*   内存压缩技术

### 内存缓存系统

*   内存替换策略: LRU, LFU
*   预取技术

### 内存数据处理系统

*   Spark

## 混合内存计算

铁电存储器
(ferroelectric random accessmemory,简称 FeRAM)[95]、相变存储器(phase change memory,简称 PCM)[9698]、电阻
存储器(resistive randomaccess memory,简称 RRAM)

# Hadoop Thesis Notes

## Google File System

```refer
Ghemawat S, Gobioff H, Leung S T. The Google file system[C]//Proceedings of the Twenty-Fourth ACM Symposium on Operating Systems Principles. ACM, 2003, 37(5): 29-43.
```

### 基本设计

*   迅速地侦测、冗余并恢复失效的组件
*   存储一定数量的大文件
*   大规模的流式读取
*   小规模的随机读取
*   大规模的、顺序的、数据追加方式的写操作
*   小规模的随机位置写入操作
*   高性能的稳定网络带宽远比低延迟重要

*   客户端和 Master 节点的通信只获取元数据，所有的数据操作都是由客户端直接和 Chunk 服务器进行交互的
*   无论是客户端还是 Chunk 服务器都不需要缓存文件数据
*   出于可靠性的考虑, 每个块都会复制到多个块服务器上.缺省情况下, 我们使用 3 个存储复制节点.

#### 元数据

Master 节点主要存储三种类型元数据 (保存在内存中, 轮询 Chunk 服务器/周期心跳信息来更新元数据):

*   文件和 Chunk 的命名空间
*   文件和 Chunk 的对应关系
*   每个 Chunk 副本的存放地点

元数据操作日志:

*   先写日志再响应请求
*   Checkpoint 文件

#### Chunk 尺寸

较大的 Chunk 尺寸:

*   降低元数据读取工作和缓存工作成本
*   多次操作集中在少数 Chunk 上

#### 一致性模型

##### Chunk 副本修改一致性

*   对 Chunk 的所有副本的修改操作顺序一致
*   根据版本号判断 Chunk 是否失效 (未成功进行某次修改), 使用垃圾收集系统回收失效 Chunk

当某个 Chunk 副本失效, 称为不可用; 当所有 Chunk 副本失效, 称为损坏

### 系统交互

原则: 减小 Master 负担

#### 租约机制

Master 为某个 Chunk 建立租约, 称为主 Chunk: 修改顺序由主 Chunk 进行管理, 从而减小 Master 管理负担:

*   Client 与 Master 通信, 获取主 Chunk 信息
*   Client 将数据传至所有 Chunk 副本
*   Client 将操作发送至主 Chunk
*   主 Chunk 为所有操作分配连续的序列号, 所有副本按照序列号顺序执行操作, 并在完成后回复主 Chunk
*   主 Chunk 回复 Client

实际上, 客户端通常会在一次请求中查询多个 Chunk 信息, Master 节点的回应也可能包含了紧跟着这些被请求的 Chunk 后面的 Chunk 的信息.
在实际应用中, 这些额外的信息在没有任何代价的情况下, 避免了客户端和 Master 节点未来可能会发生的几次通讯.

#### 数据流

链式顺序最近推送原则: 推送至距离最近 (IP 地址) 的一个下游

#### 原子追加操作

GFS并不保证 Chunk 的所有副本在字节级别是完全一致的。它只保证数据作为一个整体原子的被至少写入一次

#### 快照

copy-on-write 技术实现快照

当 Master 节点收到一个快照请求，它首先取消作快照的文件的所有 Chunk 的租约。这个措施保证了后续对这些 Chunk 的写操作都必须与 Master 交互以找到租约持有者

Master 节点通过复制源文件或者目录的元数据的方式，把这条日志记录的变化反映到保存在内存的状态中。新创建的快照文件和源文件指向完全相同的 Chunk 地址。

确保数据在本地复制而不是通过网络复制

### Master 节点

Master 节点管理所有的文件系统元数据

元数据包括:

*   名字空间
*   访问控制信息
*   文件和 Chunk 的映射信息
*   当前 Chunk 的位置信息

Master 节点基本工作:

*   执行所有的名称空间操作
*   管理着整个系统里所有 Chunk 的副本：决定 Chunk的存储位置，创建新 Chunk 和它的副本，协调各种各样的系统活动以保证 Chunk 被完全复制，在所有的 Chunk
服务器之间的进行负载均衡，回收不再使用的存储空间

#### 名称空间锁

每一个操作都获取一个目录名的上的读取锁和文件名上的写入锁。目录名的读取锁足以的防止目录被删除、改名以及被快照。文件名的写入锁序列化文件创建操作，确保不会多次创建同名的文件。

#### Chunk 副本平衡

*   平衡硬盘使用率
*   限制同一台 Chunk 服务器上的正在进行的克隆操作的数量
*   在机架间分布副本

#### 垃圾回收

*   回收过期(版本号不一致)和损坏的 Chunk.
*   GFS 空间回收采用惰性的策略: 只在文件和 Chunk 级的常规垃圾收集时进行真正的删除操作.
*   垃圾回收把存储空间的回收操作合并到 Master 节点规律性的后台活动中(例行扫描和与 Chunk 服务器握手).

### 容错性

*   快速自启动
*   Chunk 副本
*   Master 服务器副本
*   Chunk Checksum 检验数据完整性

## MapReduce

```refer
Dean J, Ghemawat S. MapReduce: simplified data processing on large clusters[C]//Proceedings of the 6th symposium on Operating systems design and implementation. USENIX Association, 2004: 137-149.
```

*   分割输入数据 (数据分布)
*   集群调度 (负载均衡)
*   错误处理 (容错)
*   集群通信

1. 用户程序首先调用的 MapReduce 库将输入文件分成 M 个数据片度，每个数据片段的大小一般从
16MB 到 64MB(可以通过可选的参数来控制每个数据片段的大小)。然后用户程序在机群中创建大量
的程序副本。
2. 这些程序副本中的有一个特殊的程序–master。副本中其它的程序都是 worker 程序，由 master 分配
任务。有 M 个 Map 任务和 R 个 Reduce 任务将被分配，master 将一个 Map 任务或 Reduce 任务分配
给一个空闲的 worker。
3. 被分配了 map 任务的 worker 程序读取相关的输入数据片段，从输入的数据片段中解析出 key/value
pair，然后把 key/value pair 传递给用户自定义的 Map 函数，由 Map 函数生成并输出的中间 key/value
pair，并缓存在内存中。
4. 缓存中的 key/value pair 通过分区函数分成 R 个区域，之后周期性的写入到本地磁盘上。缓存的
key/value pair 在本地磁盘上的存储位置将被回传给 master，由 master 负责把这些存储位置再传送给
Reduce worker。
5. 当 Reduce worker 程序接收到 master 程序发来的数据存储位置信息后，使用 RPC 从 Map worker 所在
主机的磁盘上读取这些缓存数据。当 Reduce worker 读取了所有的中间数据后，通过对 key 进行排序
后使得具有相同 key 值的数据聚合在一起。由于许多不同的 key 值会映射到相同的 Reduce 任务上，
因此必须进行排序。如果中间数据太大无法在内存中完成排序，那么就要在外部进行排序。
6. Reduce worker 程序遍历排序后的中间数据，对于每一个唯一的中间 key 值，Reduce worker 程序将这
个 key 值和它相关的中间 value 值的集合传递给用户自定义的 Reduce 函数。Reduce 函数的输出被追
加到所属分区的输出文件。
7. 当所有的 Map 和 Reduce 任务都完成之后，master 唤醒用户程序。在这个时候，在用户程序里的对
MapReduce 调用才返回。

## Google Bigtable

```refer
Chang F, Dean J, Ghemawat S, et al. Bigtable: A Distributed Storage System for Structured Data[C]//Proceedings of the 7th symposium on Operating systems design and implementation. USENIX Association, 2006: 205-218.
```

Bigtable 是一个稀疏的、分布式的、持久化存储的多维度排序 Map: `(row: string, column: string, time: int64) -> string`

### 数据模型


#### 行

*   对同一个行关键字的读或者写操作都是原子的
*   Bigtable 通过行关键字的字典顺序来组织数据
*   表中的每个行都可以动态分区, 每个分区叫做一个 "Tablet"
*   Tablet 是数据分布和负载均衡调整的最小单位

#### 列族

*   列关键字组成的集合叫做 "列族", 列族是访问控制的基本单位
*   存放在同一列族下的所有数据通常都属于同一个类型
*   访问控制、磁盘和内存的使用统计都是在列族层面进行的

#### 时间戳

*   在 Bigtable 中, 表的每一个数据项都可以包含同一份数据的不同版本
*   不同版本的数据通过时间戳来索引

### 依赖

#### SSTable

*   BigTable 内部存储数据的文件是 Google SSTable 格式的, SSTable 是一个持久化的、排序的、不可更改的Map 结构
*   SSTable 使用块索引（通常存储在 SSTable 的最后）来定位数据块
*   在打开 SSTable 的时候，索引被加载到内存
*   每次查找都可以通过一次磁盘搜索完成: 首先使用二分查找法在内存中的索引里找到数据块的位置，然后再从硬盘读取相应的数据块

#### Chubby

Bigtable 使用 Chubby 完成以下的几个任务：

1. 确保在任何给定的时间内最多只有一个活动的 Master 副本
2. 存储 BigTable 数据的自引导指令的位置
3. 查找 Tablet 服务器，以及在 Tablet 服务器失效时进行善后
4. 存储 BigTable 的模式信息(每张表的列族信息)
5. 存储访问控制列表

### 组件

Bigtable 包括了三个主要的组件: 链接到客户程序中的库、一个 Master 服务器和多个 Tablet 服务器

#### Master 服务器

Master 服务器主要负责以下工作：

*   为 Tablet 服务器分配 Tablets
*   检测新加入的或者过期失效的 Table 服务器
*   对 Tablet 服务器进行负载均衡
*   对保存在 GFS 上的文件进行垃圾收集
*   处理对模式的相关修改操作, 例如建立表和列族

#### Tablet 服务器

*   每个 Tablet 服务器都管理一个 Tablet 的集合（通常每个服务器有大约数十个至上千个 Tablet）
*   每个 Tablet服务器负责处理它所加载的 Tablet 的读写操作
*   在 Tablets 过大时，对其进行分割

### 实现

#### 定位

*   使用三层 B+ 树结构存储 Tablets 的位置信息 (位于哪个 Tablet 服务器上)
*   MetaTablet中 的 row key 是通过 table id 和 end row 来计算出的: 因为 row 是有序的, 所以在读取 row/column 型的数据时可以利用到有序性

#### 服务

*   当写入表时, 先写日志, 再写入内存中的 memtable, 当其大小达到阈值后, 写入 GFS 中 (以 SSTable 存储)
*   利用 Log-Structured Merge Tree 来表示 memtable 和 SSTable 之间的关系 (内存数据滚动合并至磁盘)
*   对于一致性的处理，将 SSTable 不可改变化，然后对 memtable 使用 copy-on-write 技术实现读写并行

# Unikernel

小、简单、安全、高效

Fast, Small, Secure Workloads

Containers are Smaller and Faster, but Security is Still an Issue

SELinux: 在一个开启了 SELinux 的内核中, 可能存在 10 万条以上的安全策略, 有时无法正常启动服务 (缺少权限)

## Unikernels

```refer
Pavlicek R C. Unikernels: beyond containers to the next generation of cloud[M]. " O'Reilly Media, Inc.", 2017.
```

Unikernels do at compile time what standard programs do at runtime:

*   Most unikernels use a specialized compiling system that compiles in the low-level functions the developer has selected
*   The code for these low-level functions is compiled directly into the application executable through a library operating system
*   library operating system: a special collection of libraries that provides needed operating system functions in a compilable format

Security:

*   There is no command shell to leverage
*   There are no utilities to co-opt
*   There are no unused device drivers or unused libraries to attack
*   There are no password files or authorization information present
*   There are no connections to machines and databases not needed by the application

## libOS

*   implement drivers for the virtual hardware devices provided by the hypervisor
*   create the protocol libraries to replace the services of a traditional OS

the kernel is event-driven via an I/O loop that polls Xen devices

## OPAM

modern modular, functional and type-safe programming language

```sh
$ opam install
$ opam update -u
$ opam switch
$ opam remote
$ opam depext
```


## MirageOS

the current release contains clean-slate libraries for TLS, TCP/IP, DNS, Xen network and storage device drivers], HTTP, and other common Internet protocols

# Memory Management

## Virtual Segmentation

在大内存系统中, 传统的段页式虚存管理技术存在性能问题 (TLBs 的有限性能):

*   由于 TLBs 不再能覆盖工作集 (working sets: 指活动频繁的页表集), 导致 TLB 缺失 (TLB misses) 急剧上升
*   在传统内存层次中, 利用 TLBs 完成地址翻译后 (虚拟地址 -> 物理地址), 需要从 L1 Cache 开始进行缓存标签匹配, 查看所需内存数据是否已在缓存上。因此，随意地增大 TLB 的大小以降低 TLB 缺失的方法会影响到所有内存操作的性能
