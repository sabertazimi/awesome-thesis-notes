# Graph Processing Notes

<!-- TOC -->

- [Graph Processing Notes](#graph-processing-notes)
  - [Graph System](#graph-system)
  - [NUMA Architecture](#numa-architecture)
    - [CPU Schedule](#cpu-schedule)
    - [Memory Schedule](#memory-schedule)
  - [Dataset](#dataset)
  - [Tools](#tools)
    - [Concurrency Lib](#concurrency-lib)
    - [Perf Tools](#perf-tools)
    - [Hardware Performance Counter](#hardware-performance-counter)
    - [Parallel Programming](#parallel-programming)
    - [Other Libs](#other-libs)
    - [DRAMSim2](#dramsim2)
    - [GCC](#gcc)
      - [strict-alias warnings](#strict-alias-warnings)
    - [Time Stamp Counter](#time-stamp-counter)
      - [RDTSC](#rdtsc)
      - [clock_gettime](#clock_gettime)
    - [NUMA Tool](#numa-tool)
      - [numactl](#numactl)
        - [installation](#installation)
        - [usage](#usage)
      - [numastat](#numastat)

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

## NUMA Architecture

引入了 node 和 distance:

- 对于 CPU 和 Memory 这两种最宝贵的硬件资源,
  NUMA 用近乎严格的方式划分了所属的资源组 (node), 而每个资源组内的 CPU 和 Memory 几乎相等
- 资源组的数量取决于物理 CPU 的个数
- distance 用来定义各个node之间调用资源的开销, 为资源调度优化算法提供数据支持

每个进程(或线程)都会从父进程继承NUMA策略, 并分配有一个优先node. 如果NUMA策略允许的话，进程可以调用其他node上的资源.

### CPU Schedule

- cpunodebind: 规定进程运行在某几个 node 之上
- physcpubind: 精细地规定进程运行在哪些核上

### Memory Schedule

- localalloc: 从当前node上请求分配内存
- preferred: 比较宽松地指定了一个推荐的 node 来获取内存, 如果被推荐的 node 上没有足够内存, 进程可以尝试别的 node
- membind: 可以指定若干个 node,进 程只能从这些指定的 node 上请求分配内存
- interleave: 规定进程从指定的若干个 node 上以 RR (Round Robin) 交织地请求分配内存

NUMA 默认的内存分配策略是优先在进程所在 CPU 的本地内存中分配, 会导致 CPU 节点之间内存分配不均衡.
当某个 CPU 节点的内存不足时, 会导致 swap 产生, 而不是从远程节点分配内存, 这就是 **swap insanity** 现象

## Dataset

- [LiveJournal social network](http://snap.stanford.edu/data/soc-LiveJournal1.html)
- [twitter rv](http://an.kaist.ac.kr/traces/WWW2010.html)
- [us road](http://www.dis.uniroma1.it/challenge9/download.shtml)

## Tools

### Concurrency Lib

- [Taskflow](https://github.com/cpp-taskflow/cpp-taskflow)

### Perf Tools

- [Flame Graph](https://github.com/brendangregg/FlameGraph)

### Hardware Performance Counter

- [Intel PCM](https://software.intel.com/en-us/articles/intel-performance-counter-monitor)
- [PAPI](https://www.icl.utk.edu/publications/papi-portable-interface-hardware-performance-counters)

### Parallel Programming

- [OpenMP](https://www.openmp.org)

### Other Libs

- [Lib BLAS](http://www.netlib.org/blas/)

### DRAMSim2

- [Overview of DRAMSim2’s Memory Structure](https://cinwell.wordpress.com/2013/09/25/general-overview-of-dramsim2s-memory-structure/)

### GCC

#### strict-alias warnings

for strict-aliasing warnings:

1. use a union to represent the memory need to reinterpret
2. use a reinterpret_cast, cast via `char *` at the point where reinterpret
  the memory - `char *` are defined as being able to alias anything
3. use a type which has `__attribute__((__may_alias__))`
4. turn off the aliasing assumptions globally using -fno-strict-aliasing

### Time Stamp Counter

- [definition](http://en.wikipedia.org/wiki/Time_Stamp_Counter)

#### RDTSC

#### clock_gettime

```bash
gcc *.c -o *.o ... -lrt # link with librt
```

```c
#include <time.h>

struct timespec ts;
clock_gettime(CLOCK_MONOTONIC, ts);
printf("%d %d", ts.tv_sec, ts.tv_nsec);
```

### NUMA Tool

```bash
grep -i numa /var/log/dmesg
```

#### numactl

##### installation

```bash
sudo apt install -y numactl
```

##### usage

```bash
numactl --show
numactl --hardware
numactl --interlave=all
```

#### numastat

```bash
numastat
```
