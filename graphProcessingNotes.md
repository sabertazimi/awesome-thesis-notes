# Graph Processing Notes

<!-- TOC -->

- [Graph Processing Notes](#graph-processing-notes)
  - [Graph System](#graph-system)
  - [GraphChi](#graphchi)
  - [X-stream](#x-stream)

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

## X-stream

- 以边为中心的并行单机图计算系统
- 将边划分为多个流式分区 (分区内无需排序), 顺序化地访问每个分区中的所有边
- 随机访问快速存储介质中的数据 (如顶点数据), 顺序访问慢速存储介质中的数据 (如边数据)
- `input --(run algorithm)--> output`
- 没有预处理开销
