## 磁盘性能指标

经常用使用率、饱和度、IOPS、吞吐量以及响应时间等这五个指标，来衡量磁盘性能的基本指标。

+ 使用率，是指磁盘处理 I/O 的时间百分比。过高的使用率（比如超过 80%），通常意味着磁盘 I/O 存在性能瓶颈。

+ 饱和度，是指磁盘处理 I/O 的繁忙程度。过高的饱和度，意味着磁盘存在严重的性能瓶颈。当饱和度为 100% 时，磁盘无法接受新的 I/O 请求。

+ IOPS（Input/Output Per Second），是指每秒的 I/O 请求数。

+ 吞吐量，是指每秒的 I/O 请求大小。

+ 响应时间，是指 I/O 请求从发出到收到响应的间隔时间。

## 磁盘 I/O 观测

```
[root@izbp1goz1ulmtus2vec80fz ~]# iostat -d -x 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       06/21/2020      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.08    0.01    0.20     1.23     3.90    48.56     0.00   17.49   12.49   17.75   0.89   0.02

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.99    0.00     7.92     0.00    16.00     0.00    1.00    1.00    0.00   1.00   0.10

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    1.01     0.00     4.04     8.00     0.00    0.00    0.00    0.00   0.00   0.00
```

第一列的 Device 表示磁盘设备的名字。

+ %util  ，就是我们前面提到的磁盘 I/O 使用率；
+ r/s+  w/s  ，就是 IOPS；
+ rkB/s+wkB/s ，就是吞吐量；
+ r_await+w_await ，就是响应时间。

## 进程 I/O 观测

```
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       06/21/2020      _x86_64_        (1 CPU)

11:22:14 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
11:22:15 PM     0       255      0.00     56.00      0.00  jbd2/vda1-8

11:22:15 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command

11:22:16 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
```

+ 用户 ID（UID）和进程 ID（PID）  。

+ 每秒读取的数据大小（kB_rd/s） ，单位是 KB。
+ 每秒发出的写请求数据大小（kB_wr/s） ，单位是 KB。
+ 每秒取消的写请求数据大小（kB_ccwr/s） ，单位是 KB。
+ 块 I/O 延迟（iodelay），包括等待同步块 I/O 和换入块 I/O 结束的时间，单位是时钟周期。

```
[root@izbp1goz1ulmtus2vec80fz ~]# iotop

30128 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % dockerd -H fd:// --containerd=/run/containerd/containerd.sock
30130 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % dockerd -H fd:// --containerd=/run/containerd/containerd.sock
26989 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % containerd-shim -namespace moby -workdir /var/lib/con~/containerd -runtime-root /var/run/docker/runtime-runc
  432 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [edac-poller]
  255 be/3 root        0.00 B/s    0.00 B/s  0.00 %  0.21 % [jbd2/vda1-8]
Total DISK READ :       0.00 B/s | Total DISK WRITE :       0.00 B/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                                                  
    1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % systemd --system --deserialize 22
```

Total DISK READ、Actual DISK READ表示进程的磁盘读写大小总数和磁盘真实的读写大小总数。因为缓存、缓冲区、I/O 合并等因素的影响，它们可能并不相等。