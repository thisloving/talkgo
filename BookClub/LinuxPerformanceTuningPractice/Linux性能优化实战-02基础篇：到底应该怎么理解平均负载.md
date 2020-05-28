

## 实验环境:

```
[root@izbp1goz1ulmtus2vec80fz tools]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.6.1810 (Core) 
Release:	7.6.1810
Codename:	Core
```

```
[root@izbp1goz1ulmtus2vec80fz tools]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Platinum 8163 CPU @ 2.50GHz
```

```
[root@izbp1goz1ulmtus2vec80fz tools]# cat /proc/meminfo
MemTotal:        1883492 kB
MemFree:          133404 kB
MemAvailable:    1511232 kB
Buffers:          203520 kB
Cached:          1234660 kB
```

## 平均负载

### 定义：

```
平均负载是指单位时间内，系统处于可运行状态和不可中断状态的平均进程数，也就是平均活跃进程数。
```

平均负载是指单位时间内，处于可运行状态和不可中断状态的进程数。所以，它不仅包括了正在使用 CPU 的进程，还包括等待 CPU 和等待 I/O 的进程。

延伸知识：

### 进程状态

- R(Running or Runable): 进程在就绪队列中,正在运行或者正在等待CPU资源运行
- D(Disk Sleep): 不可中断状态睡眠(Uninterruptible Sleep),表示进程正在和硬件交互,且交互过程不允许被其他进程或者中断打断
- Z(ZOmbie): 僵尸进程,进程实际上已经结束, 但是父进程没有回收它的资源(如进程描述符, PID等)
- S(Interruptible Sleep): 可中断睡眠状态,表示进程因为某个事件而被系统挂起.
- I(Idle): 空闲状态
- T(Stooped, Traced): 暂停或者跟踪状态

### 查看系统平均负载：

```
[leving@izbp1goz1ulmtus2vec80fz talkgo]$ uptime
 12:29:35 up 780 days, 21:02,  2 users,  load average: 0.01, 0.03, 0.05
```

0.01指的是过去1分钟的系统负载

0.03指的是过去5分钟的系统负载

0.05指的是过去15分钟的系统负载

###导致系统负载升高情形分析

先认识linux命令

mpstat -P ALL 5 : -P ALL表示监控所有CPU，5表示每5秒刷新一次数据

pidstat -u 5 1：每5秒输出一组数据，观察哪个进程%cpu很高

#### 场景一：CPU 密集型进程

```
[leving@izbp1goz1ulmtus2vec80fz talkgo]$ stress --cpu 1 --timeout 600
stress: info: [28706] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# uptime
 13:33:28 up 780 days, 22:05,  4 users,  load average: 3.15, 2.37, 1.67
```

```
//mpstat 查看 CPU 使用率的变化情况，-P ALL 表示监控所有CPU，后面数字5表示间隔5秒后输出一组数据
[root@izbp1goz1ulmtus2vec80fz tools]# mpstat -P ALL 5 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

01:39:45 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:39:50 PM  all  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
01:39:50 PM    0  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00

Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
Average:     all  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
Average:       0  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00

```

```
//哪个进程导致了 CPU 使用率为 100%
[root@izbp1goz1ulmtus2vec80fz tools]# pidstat -u 5 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

01:41:41 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
01:41:46 PM     0      2267    0.00    0.21    0.00    0.21     0  redis-server
01:41:46 PM     0     24144    0.21    0.21    0.00    0.43     0  AliYunDun
01:41:46 PM  1000     28707  100.00    0.00    0.00  100.00     0  stress
01:41:46 PM     0     29621    0.00    0.21    0.00    0.21     0  containerd

```

综上结果，cpu繁忙导致评价负载升高。另外，htop也可以动态观察cpu负载情况。

#### 场景二：I/O 密集型进程

```
// 模拟 I/O 压力
[root@izbp1goz1ulmtus2vec80fz ~]# stress-ng -i 1 --hdd 1 --timeout 600
stress-ng: info:  [29910] dispatching hogs: 1 io, 1 hdd
```

```
// 查看平均负载
[root@izbp1goz1ulmtus2vec80fz ~]# uptime
 13:51:34 up 780 days, 22:24,  5 users,  load average: 3.98, 3.05, 2.46
```

```
// 显示所有CPU的指标，iowait负载比较高
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

01:52:22 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:52:27 PM  all    0.64    0.00    9.81   89.55    0.00    0.00    0.00    0.00    0.00    0.00
01:52:27 PM    0    0.64    0.00    9.81   89.55    0.00    0.00    0.00    0.00    0.00    0.00

```

```
[root@izbp1goz1ulmtus2vec80fz tools]# pidstat -u 5 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

01:54:04 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
01:54:09 PM     0       255    0.00    0.42    0.00    0.42     0  jbd2/vda1-8
01:54:09 PM     0      2267    0.21    0.00    0.00    0.21     0  redis-server
01:54:09 PM     0     24080    0.00    0.21    0.00    0.21     0  AliYunDunUpdate
01:54:09 PM     0     24144    0.64    0.64    0.00    1.27     0  AliYunDun
01:54:09 PM     0     29621    0.21    0.00    0.00    0.21     0  containerd
01:54:09 PM     0     29912    0.00   15.04    0.00   15.04     0  stress-ng-hdd
01:54:09 PM     0     30492    0.00    1.27    0.00    1.27     0  kworker/u2:0
```

综上结果，是stress-ng导致的平均负载过高。

#### 场景三：大量进程的场景

```
//模拟 8 个进程
[root@izbp1goz1ulmtus2vec80fz ~]# stress -c 8 --timeout 600
stress: info: [30559] dispatching hogs: 8 cpu, 0 io, 0 vm, 0 hdd
```

```
//查看平均负载
[root@izbp1goz1ulmtus2vec80fz ~]# uptime
 14:01:39 up 780 days, 22:34,  5 users,  load average: 8.95, 5.78, 3.91
```

```
[root@izbp1goz1ulmtus2vec80fz tools]# pidstat -u 5 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

02:02:15 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:02:20 PM     0      2267    0.21    0.00    0.00    0.21     0  redis-server
02:02:20 PM     0     20846    0.21    0.00    0.00    0.21     0  aliyun-service
02:02:20 PM     0     24144    0.21    0.21    0.00    0.43     0  AliYunDun
02:02:20 PM     0     30668   13.89    0.00    0.00   13.89     0  stress
02:02:20 PM     0     30669   13.68    0.00    0.00   13.68     0  stress
02:02:20 PM     0     30670   13.68    0.00    0.00   13.68     0  stress
02:02:20 PM     0     30671   13.89    0.00    0.00   13.89     0  stress
02:02:20 PM     0     30672   13.68    0.00    0.00   13.68     0  stress
02:02:20 PM     0     30673   13.68    0.00    0.00   13.68     0  stress
02:02:20 PM     0     30674   13.68    0.00    0.00   13.68     0  stress
02:02:20 PM     0     30675    9.62    0.00    0.00    9.62     0  stress
```

## 总结

​		了解了平均负载定义，以及查看平均负载的指令，根据各种情形查看导致负载升高的原因。

