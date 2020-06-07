## CPU 性能指标

​        1、 首先，最容易想到的应该是 CPU 使用率，CPU 使用率描述了非空闲时间占总 CPU 时间的百分比，根据 CPU 上运行任务的不同，又被分为用户 CPU、系统 CPU、等待 I/O CPU、软中断和硬中断等。

​        2、 第二个比较容易想到的，应该是平均负载（Load Average）。

​		3、第三个，进程上下文切换。包括：无法获取资源而导致的自愿上下文切换、被系统强制调度导致的非自愿上下文切换。

​		4、CPU 缓存的命中率。

## 性能工具

​		1、 uptime， 查看了系统的平均负载。

​		2、mpstat ，观察每个 CPU  的使用情况。

​		3、pidstat  ，观察每个进程 CPU 的使用情况。

​		4、vmstat  ，查看系统的上下文切换次数和中断次数。此时，pidstat ，观察了进程的自愿上下文切换和非自愿上下文切换情况（-w参数）。

​		5、sysbench，观察线程的上下文切换情况，上下文切换次数。

​		6、perf top（perf  record和perf report)观察系统函数调用占有情况。

​		7、execsnoop。观察短时进程的创建。 

​		8、cat /proc/softirqs 和 cat /proc/interrupts 查看中断情况。

​		9、dstat，会收集-cpu-、-disk-、-net-、－paging-、-system-的数据

## 如何迅速分析 CPU 的性能瓶颈

​		为了缩小排查范围，通常这3个工具top、vmstat 和 pidstat先入手。

本章主要是回忆，会议之前的信息工具的使用。