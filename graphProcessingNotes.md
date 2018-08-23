# Graph Processing Notes

<!-- TOC -->

- [Graph Processing Notes](#graph-processing-notes)
  - [Graph System](#graph-system)
  - [GraphChi](#graphchi)
    - [GraphChi shortcoming](#graphchi-shortcoming)
  - [X-stream](#x-stream)
    - [X-stream shortcoming](#x-stream-shortcoming)
  - [GridGraph](#gridgraph)
    - [Motivation](#motivation)
    - [Solution](#solution)
    - [Grid Representation](#grid-representation)
    - [Dual Sliding Windows](#dual-sliding-windows)
    - [Selective Scheduling](#selective-scheduling)
    - [2-Level Hierarchical Partitioning](#2-level-hierarchical-partitioning)
    - [GridGraph Shortcoming](#gridgraph-shortcoming)

<!-- /TOC -->

## Graph System

|分类|In-memory|Out-of-core|
|:---------:|:--------------------:|:--------------------:|
|Single machine|Ligra|GraphChi|
||Polymer|X-Stream|
||Galois|GridGraph|
|||Mosaic|
|Distributed|Pregel|Chaos|
||PowerGraph||
||PowerLyra||
||Gemini||

## GraphChi

- 以顶点为中心的并行单机图计算系统
- 将连续顶点集划分为 P 个 interval， 每个 interval 对应一个 shard, 存储着所有入边 (即一个 shard 对应一个子图, 边以源点排序保存)
- 利用 PSW (parallel sliding window) 每次访问一个 shard 中所有边与其他 shards 中的出边
- `input --(create shards)--> shards --(run algorithm)--> output`
- 尽管需要进行预处理 (创建 shards, 排序等), 但对于需要长期运行的系统而言, 这仅仅需要进行非常少的次数 (one time cost)

### GraphChi shortcoming

- 构建shard是需要对边按源顶点排序，这样耗费了大量的预处理时间，PWS对计算密集型的算法更有利
- 另外在构建子图时出现大量的随机访存现象，通过顺序地更新子图内有共享边顶点来避免数据争用问题

## X-stream

- 以边为中心的并行单机图计算系统
- 将边划分为多个流式分区 (分区内无需排序), 顺序化地访问每个分区中的所有边
- 随机访问快速存储介质中的数据 (如顶点数据), 顺序访问慢速存储介质中的数据 (如边数据)
- `input --(run algorithm)--> output`
- 没有预处理开销

### X-stream shortcoming

- X-Stream在计算过程中，每轮迭代产生的更新集非常庞大，接近于边的数量级
- 需要对更新集中的边进行shuffle操作
- 缺乏选择调度机制，产生了大量的无用计算

## GridGraph

### Motivation

基于 X-stream, 能否进行实时更新, 从而进一步减少 I/O 开销:

将 scatter 阶段的顺序化写入与 gather 阶段的顺序化读出进行一定程度上地合并, 即合并 updates 操作

### Solution

- 在进行流式分区时, 保证 scatter 阶段的源顶点与 gather 阶段的目的顶点的局部性 (使其位于同一个 streaming partion)
- 利用 Grid Representation 可以达到这一目的: 两级结构的分区 (2-level hierarchical partitioning) + 双滑动窗口 (Dual sliding windows)

### Grid Representation

- 将顶点划分为 P 个相等的 chunks
- 将边划分为 P x P 个 blocks: 边源顶点所在的chunk决定其在网格中的行，边目的顶点所在的chunk决定其在网格中的列

第 n 行中的边的源顶点应为 chunk-n 中的顶点, 第 n 列中的边的目的顶点应为 chunk-n 中的顶点: 第 (m, n) 个 block 中的边的源顶点应为 chunk-m 中的顶点, 目的顶点应为 chunk-n 中的顶点

### Dual Sliding Windows

- 以列优先的顺序逐块遍历 edge blocks (自上至下, 自左至右)
- 以列优先的顺序遍历时, 可以减少写带宽 (gather 阶段的 updates)

### Selective Scheduling

跳过不具有活跃边的 blocks, 减少不必要的 I/O

### 2-Level Hierarchical Partitioning

将每个 Block 进行进一步的 Grid 划分

### GridGraph Shortcoming

- 折线式的边 block 遍历策略不能达到最大化的Cache/Memory命中率
