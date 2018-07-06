<!-- TOC -->

- [Conference List](#conference-list)
- [In-memory Computing Thesis Notes](#in-memory-computing-thesis-notes)
    - [单节点内存计算](#单节点内存计算)
        - [内存存储系统](#内存存储系统)
        - [内存数据处理系统](#内存数据处理系统)
        - [内存压缩技术](#内存压缩技术)
        - [提升 I/O 访问效率](#提升-io-访问效率)
    - [分布式内存计算](#分布式内存计算)
        - [内存存储系统](#内存存储系统-1)
        - [内存缓存系统](#内存缓存系统)
        - [内存数据处理系统](#内存数据处理系统-1)
    - [混合内存计算](#混合内存计算)
- [Hadoop Thesis Notes](#hadoop-thesis-notes)
    - [Google File System](#google-file-system)
        - [基本设计](#基本设计)
            - [元数据](#元数据)
            - [Chunk 尺寸](#chunk-尺寸)
            - [一致性模型](#一致性模型)
                - [Chunk 副本修改一致性](#chunk-副本修改一致性)
        - [系统交互](#系统交互)
            - [租约机制](#租约机制)
            - [数据流](#数据流)
            - [原子追加操作](#原子追加操作)
            - [快照](#快照)
        - [Master 节点](#master-节点)
            - [名称空间锁](#名称空间锁)
            - [Chunk 副本平衡](#chunk-副本平衡)
            - [垃圾回收](#垃圾回收)
        - [容错性](#容错性)
    - [MapReduce](#mapreduce)
    - [Google Bigtable](#google-bigtable)
        - [数据模型](#数据模型)
            - [行](#行)
            - [列族](#列族)
            - [时间戳](#时间戳)
        - [依赖](#依赖)
            - [SSTable](#sstable)
            - [Chubby](#chubby)
        - [组件](#组件)
            - [Master 服务器](#master-服务器)
            - [Tablet 服务器](#tablet-服务器)
        - [实现](#实现)
            - [定位](#定位)
            - [服务](#服务)
- [Unikernel](#unikernel)
    - [Unikernels](#unikernels)
        - [Security](#security)
        - [Debug](#debug)
        - [Limits](#limits)
        - [Ecosystem](#ecosystem)
            - [ClickOS](#clickos)
            - [Rumprun](#rumprun)
            - [OSv](#osv)
            - [Mini-OS](#mini-os)
            - [Xen](#xen)
            - [Unik](#unik)
    - [libOS](#libos)
    - [OPAM](#opam)
    - [MirageOS](#mirageos)
        - [Installation](#installation)
        - [Development](#development)
        - [Introduction](#introduction)
        - [Architecture](#architecture)
        - [Use of OCaml](#use-of-ocaml)
        - [PVBoot Library](#pvboot-library)
        - [Heap Management and Concurrency (Language Runtime)](#heap-management-and-concurrency-language-runtime)
            - [Thread Flow](#thread-flow)
        - [Device Drivers](#device-drivers)
        - [I/O Stack](#io-stack)
- [Memory Management](#memory-management)
    - [Direct Segment](#direct-segment)
        - [Large Pages Shortcoming](#large-pages-shortcoming)
        - [Big-memory workload analysis](#big-memory-workload-analysis)
        - [Hardware Support](#hardware-support)
        - [Software Support](#software-support)
        - [Virtual Machines with direct-segment](#virtual-machines-with-direct-segment)
        - [Results](#results)
    - [Redundant Memory Mappings (RMM)](#redundant-memory-mappings-rmm)
        - [Old Methods](#old-methods)
        - [multipage mapping](#multipage-mapping)
        - [transparent huge pages](#transparent-huge-pages)
        - [direct segment](#direct-segment)
        - [New Methods](#new-methods)
        - [RMM Benefits](#rmm-benefits)
        - [Range Translation](#range-translation)
        - [Range Table](#range-table)
        - [Contiguous Memory Allocation](#contiguous-memory-allocation)

<!-- /TOC -->

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

### Security

*   There is no command shell to leverage
*   There are no utilities to co-opt
*   There are no unused device drivers or unused libraries to attack
*   There are no password files or authorization information present
*   There are no connections to machines and databases not needed by the application

### Debug

Since there is no operating system environment on a unikernel production VM, there are no tools, no debuggers, no shell access with which someone can probe a failure on a deployed program

Instead, all that can be done is to engineer the executable to log events and data so that the failure might be reconstructed on a development system, which still has access to all the debugging tools needed

People in our industry have successfully debugged failures of complex software for years without direct access to production systems, and I see no reason why they will fail to do so now

### Limits

*   Single Process (but Multiple Threads)
*   Single User
*   Limited Debugging
*   Impoverished Library Ecosystem

### Ecosystem

#### ClickOS

network function virtualization (NFV) devices

#### Rumprun

compile most of the programs found on a Linux or Unix-like operating system as unikernels

#### OSv

It is unique among existing unikernels in that it provides a general-purpose unikernel base that can accept just about any program that can run in a single process (multiple threads are allowed, but multiple processes are not)

As a result, OSv is a “fat” unikernel: the results are measured in megabytes, rather than kilobytes. 

#### Mini-OS

Mini-OS is a tiny OS kernel distributed with the Xen Project Hypervisor sources:

*   used as operating system for stub domains that are used for Dom0 Disaggregation
*   used as a basis for development of Unikernels, having been instrumental in the formation of multiple examples including ClickOS and Rump kernels

#### Xen

Its paravirtualization capabilities allow unikernels to have a very small and efficient footprint interfacing with devices.
This is one reason why the Xen Project has been at the forefront of unikernel development. 

Xen team has been consciously reworking the hypervisor’s capabilities so it can handle a future state where 2,000 or 3,000 simultaneous unikernel VMs may need to coexist on a single hardware host server.
Currently, the hypervisor can handle about 1,000 unikernels simultaneously before scaling becomes nonlinear. The development work continues to improve unikernel support in each release.

#### Unik

*   UniK is an open source tool written in Go for compiling applications into unikernels and deploying those unikernels
*   UniK utilizes a simple Docker-like command-line interface, making developing on unikernels as easy as developing on containers

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

*   **Unikernels**: specialised, sealed, singlepurpose libOS VMs that run directly on the hypervisor
*   A libOS is structured very differently from a conventional OS: all services, from the scheduler to the device drivers to the network stack, are implemented as libraries linked directly with the application
*    Coupled with the choice of a modern statically type-safe language for implementation, this affords configuration, performance and security benefits to unikernels
*   the current release contains clean-slate libraries for TLS, TCP/IP, DNS, Xen network and storage device drivers], HTTP, and other common Internet protocols

```cl
Madhavapeddy A, Mortier R, Rotsos C, et al. Unikernels: Library operating systems for the cloud[C]//Proceedings of the 18th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). ACM, 2013: 461-472.
```

### Installation

```sh
opam init
opam remote
ocaml -version
opam switch 4.04.2
eval `opam config env`
opam list
opam install mirage
```

```sh
mirage --help
mirage configure --xen
make depend
make
sudo xl create -c console.xl
strip console.xen
bzip2 -9 console.xen
```

### Development

```sh
$ opam source repo.name --dev-repo --pin
```

### Introduction

We treat the final VM image as a single-purpose appliance rather than a general-purpose system by stripping away functionality at compile-time:

*   the unikernel approach to providing sealed single-purpose appliances, particularly suitable for providing cloud services
*   evaluation of a complete implementation of these techniques using a functional programming language (OCaml), showing that the benefits of type safety need not damage performance
*   libraries and language extensions supporting systems programming in OCaml 

The entire software stack of system libraries, language runtime, and applications is compiled into a single bootable VM image that runs directly on a standard hypervisor:

*   (SOSP'95 Exokernel) By targeting a standard hypervisor, unikernels avoid the hardware compatibility problems encountered by traditional library OSs
*   (ASPLOS'11 Drawbridge) By eschewing backward compatibility, unikernels address cloud services rather than desktop applications
*   (EuroSys'06/07 Singularity) By targeting the commodity cloud with a library OS, unikernels can provide greater performance and improved security compared to Singularity
*   (VEE'07 Libra) unikernels are more highly-specialised single-purpose appliance VMs that directly integrate communication protocols
*   We find sacrificing source-level backward compatibility allows us to increase performance while significantly improving the security of external-facing cloud services
*   We retain compatibility with external systems via standard network protocols such as TCP/IP

Our key insight is that the hypervisor (SOSP'03 Xen) provides a virtual hardware abstraction that can be scaled dynamically – both vertically by adding memory and vCPUs, and horizontally by spawning more VMs, providing an excellent target for library operating systems (libOSs) (LibOSs have never been widely deployed due to the difficulty of supporting a sufficient range of real-world hardware)

### Architecture

*   Unikernels take a different approach, by integrating configuration into the compilation process, hence benefiting from static analysis tools and the compiler’s type-checker (OCaml), extensive dead-code elimination (**eliminate** unnecessary features from the final VM, **compactness** and reduce remote attack surface)
*   unikernels use the hypervisor as the sole unit of isolation and let applications trust external entities via protocol libraries such as SSL or SSH
*   eschew source-level **compatibility** (for performance): components communicate using type-safe, efficient implementations of standard network protocols; existing non-OCaml code can be encapsulated in separate VMs and communicated with via message-passing
*   no longer requires userspace processes: the virtual address space can be simplified into a **single-address** space model
*   completely preventing code injection attacks: as part of its startof-day initialisation, the unikernel establishes a set of page tables
in which no page is both writable and executable (write XOR execute): This approach does mean that a running VM **cannot expand its heap**

### Use of OCaml

*   (ICFP'10) a full-fledged systems programming language with a flexible programming model that supports functional, imperative and object-oriented programming
*   OCaml has a simple yet highperformance runtime making it an ideal platform for experimenting with the unikernel abstraction that interfaces the runtime with Xen
*    static typing eliminates type information at compile-time while retaining all the benefits of type-safety
*    (SOSP'03/11) the open-source Xen Cloud Platform and critical system components are
implemented in OCaml, making integration straightforward

### PVBoot Library

PVBoot provides start-of-day support to initialise a VM with one virtual CPU and Xen event channels, and jump to an **entry function**

PVBoot provides two memory page allocators, one slab and one extent:

*   The slab allocator is used to support the C code in the runtime
*   The extent allocator reserves a contiguous area of virtual memory which it manipulates in 2MB chunks (permitting the mapping of x86 64 superpages)
*   Memory regions are statically assigned roles, e.g., the garbage collected heap or I/O data pages

### Heap Management and Concurrency (Language Runtime)

*   The text and data segments contain the OCaml runtime, linked against PVBoot: this is the only address space available in the kernel
*    a normal userspace garbage collector maintains a page table to track allocated heap chunks
*   Mirage unikernels avoid ASR at runtime, and guarantee a **contiguous
virtual address space**, simplifying runtime memory management
*   (ML'08 Lwt) Mirage integrates the Lwt cooperative threading library

#### Thread Flow

*   The application’s main thread is launched immediately after boot and the VM shuts down when it returns
*   Mirage provides an evaluator that uses domainpoll to listen for events and wake up lightweight threads. The VM is thus either executing OCaml code or blocked, with no internal preemption or asynchronous interrupts
*   The main thread repeatedly executes until it completes or throws an exception, and the domain subsequently shuts down with the VM exit code matching the thread return value

### Device Drivers

Data arrives to both the network and storage stacks as a stream of discrete packets. Mirage bridges the gap between packets and streams by using channel iteratees that map functions over infinite streams of packets to produce a typed stream

### I/O Stack

# Memory Management

## Direct Segment

```cl
Basu A, Gandhi J, Chang J, et al. Efficient virtual memory for big memory servers[C]//Proceedings of the 40th Annual International Symposium on Computer Architecture (ISCA). ACM, 2013: 237-248.
```

In light of the high cost of page-based virtual memory and its significant mismatch to “big-memory” application needs, we propose mapping part of a process’s linear virtual address with a direct segment rather than pages

Mapping part of a process’s linear virtual address space with a direct segment, while page mapping the rest of the virtual address space:

*   retains a standard linear virtual address space
*   is not overlaid on top of paging
*   coexists with paging of other virtual addresses
*   a one-time fixed-cost solution for any size memory

### Large Pages Shortcoming

*   Being a cache, TLBs are reliant on memory-access locality to be effective and it can be a mismatch for future big-memory workloads with poor locality
*   efficient TLB support for multiple page sizes is difficult
*   large page sizes are often few and far apart (not enough for big-memory workload)

### Big-memory workload analysis

*   While address translation can be accelerated by TLB hits, misses are costly, taking up to 100s of cycles
*   A sparse memory access pattern can result in more misses with fewer TLB entries

For the majority of their address space, big-memory workloads do not require:

*   swapping
*   fragmentation mitigation
*   fine-grained per-page protection

*   Big-memory workloads pay a cost of pagebased virtual memory: substantial performance lost to TLB misses
*   Big-memory workloads are long-running programs, receive little benefit from virtual memory optimizations whose primary goal is to allow quick program startup, such as demand paging
*   Big-memory workloads cosume almost all memory resources, are sized to match memory capacity
*   Big-memory workloads isolate from other services, have one (or a few) primary process(es)

### Hardware Support

direct segments add three registers per core as follows:

*   BASE holds the start address of the contiguous virtual address range mapped through direct segment
*   LIMIT holds the end address of the virtual address range mapped through direct segment
*   OFFSET holds the start address of direct segment’s backing contiguous physical memory minus the value in BASE.

Direct segments are aligned to the base page size, so page offset bits are omitted from these registers (e.g., 12 bits for 4KB pages)

Without real direct-segment hardware, we emulate directsegment functionalities using 4KB pages

More specifically, we modify Linux’s page fault handler so that on a page fault within the primary region it calculates the corresponding physical address from the faulting virtual page number

If VAfault is the 4KB page-aligned virtual address of a faulting page, then our modified page-fault handler first checks if BASE ≤ VAfault < LIMIT. If so, the handler adds a mapping from VAfault to VAfault + OFFSET to the page table

### Software Support

The prototype implementation is simplified by assuming that only one process uses a direct segment at any time (called the primary process)

*   A primary region is a contiguous range of virtual addresses in a process’s address space with uniform **read-write** access permission
*   fine-grain protection, sparse allocation, swapping, and demand paging are not guaranteed for memory allocated within the primary region
*   managing physical memory: create contiguous physical memory dynamically through periodic memory compaction
*   managing direct-segment registers: When the OS dispatches a thread; it loads the BASE, LIMIT, and OFFSET values from the PCB
*   Growing and shrinking direct segment (by updating direct-segment registers and page table entries)


### Virtual Machines with direct-segment

In a virtualized environment the memory accesses goes through two levels of address translations:

1. guest virtual address (gVA) to guest physical address (gPA)
2. guest physical address (gPA) to system physical address (sPA)

### Results

In x86-64, page table entries (PTEs) have a set of reserved bits (41-51 in our test platform) that cause a trap when loaded into the TLB

## Redundant Memory Mappings (RMM)

```cl
V. Karakostas, J. Gandhi, F. Ayar, A. Cristal, M. D. Hill, K. S. McKinley, M. Nemirovsky, M. M. Swift, and O. Ünsal, “Redundant Memory Mappings for Fast Access to Large Memories,” in Proceedings of the 42Nd Annual International Symposium on Computer Architecture (ISCA), New York, NY, USA, 2015, pp. 66–78
```

在大内存系统中, 传统的段页式虚存管理技术存在性能问题 (TLBs 的有限性能):

*   由于 TLBs 不再能覆盖工作集 (working sets: 指活动频繁的页表集), 导致 TLB 缺失 (TLB misses) 急剧上升
*   在传统内存层次中, 利用 TLBs 完成地址转换后 (虚拟地址 -> 物理地址), 需要从 L1 Cache 开始进行缓存标签匹配, 查看所需内存数据是否已在缓存上。因此，随意地增大 TLB 的大小以降低 TLB 缺失的方法会影响到所有内存操作的性能 (遍历 TLB 表耗时增大)

The amount of memory accessible from the TLB or The working set of each process is stored in the TLB: 
    TLB Reach = (TLB Size) X (Page Size)

### Old Methods

*   clustered TLBs/multipage mappings: mapping multiple pages per page table entry
*   transparent huge pages/superpaging: 使得每个页表项覆盖更大的内存; 但受限于架构, 页面大小最大为 1GB
*   direct segment: 将一个进程所需的虚拟内存直接映射到一个大段中; 但不利用操作系统的动态内存分配

### multipage mapping

*   pack multiple page table entries(PTEs) into a single TLB entry
*   they pack only a small multiple of translations per entry, which limits their potential to reduce page-walks for large working sets

### transparent huge pages

*   memory should be size-aligned and contiguous
*   many commodity processors provide limited numbers of large page TLB entries

### direct segment

*   applications explicitly allocate a direct segment during `startup`
*   OS can reserve a single `large contiguous range` of physical memory for a segment

### New Methods

two key challenges:

*   The address space of a process can be decomposed  into  many  chunks  of  variable  length  segments, which can be allocated dynamically
*   address  translation  must be off the critical path: to support complicated many segment searches, address translation may take much longer compared to traditional fixed-sized page-based TLB lookups

changes:

*   hardware: range TLB, CR-RT (Control Register - Range Translation)
*   software: process creation, memory allocation, range table management

### RMM Benefits

*   `transparent` to applications
*   maps multiple ranges with `no size-alignment restrictions`
*   each range contains an `unrestricted amount of memory`
*   RMM has the potential to map more than 99% of memory for all workloads with 50 or fewer range translations 
*   the range TLB achieves extremely high hit rates, eliminating the vast majority of costly page-walks compared to virtual memory systems that use paging alone

RMM performance depends on the range TLB achieving a high hit ratio with few entries

### Range Translation

Range translations are only base-page-aligned and redundant to paging: the page table still maps the entire virtual address space

*   hardware: range TLBs
*   software: range tables
*   OS: change lazy demand page allocation to eager paging, `eager paging allocation`

Eager paging instantiates pages in physical memory at `allocation request time`, rather than at `first-access time` as with demand paging

### Range Table

*   when miss: 及时填补 PTE 以使访存操作正常执行, 将 Range TLB 更新移出访存关键路径 (后台更新)
*   PTE 加入 range bit: 表示当前页是某个 Range 的一部分, 及时更新 Range TLB 和 Range Table
*   利用 CR-RT 寄存器进行 Range Table 查找: OS handler 可以进行 Range Table 查找, 以简化硬件设计
*   修改 INVLPG 指令： 使其同时无效化 Range TLB

### Contiguous Memory Allocation

*   one motivation for demand paging was to limit unnecessary swapping in multiprogrammed workloads, which modern large memories make less common
*   RMM trades increased memory for better performance, a common tradeoff when memory is cheap and plentiful
