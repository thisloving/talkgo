## 查看cpu上下文切换

```
// vmstat常用来分析 CPU 上下文切换和中断的次数。
[root@izbp1goz1ulmtus2vec80fz tools]# vmstat 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 3  0      0 1293248  21184 361128    0    0     1     3    0    1  0  0 99  0  0
```

cs（context switch）是每秒上下文切换的次数。

in（interrupt）则是每秒中断的次数。

r（Running or Runnable）是就绪队列的长度，也就是正在运行和等待 CPU 的进程数。

b（Blocked）则是处于不可中断睡眠状态的进程数。

```
// pidstat 查看每个进程上下文切换的情况
[root@izbp1goz1ulmtus2vec80fz ~]# pidstat -w 5
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

08:01:27 PM   UID       PID   cswch/s nvcswch/s  Command
08:01:32 PM     0         1      0.80      0.00  systemd
08:01:32 PM     0         3      1.20      0.00  ksoftirqd/0
08:01:32 PM     0         9     47.49      0.00  rcu_sched
08:01:32 PM     0        10      0.20      0.00  watchdog/0
08:01:32 PM     0        27      0.20      0.00  khugepaged
08:01:32 PM     0      2117     10.02      0.00  redis-server
08:01:32 PM     0      2267     10.02      0.00  redis-server
08:01:32 PM     0      9949      0.40      0.00  top
08:01:32 PM     0     15826      0.20      0.00  vmstat
08:01:32 PM     0     16000      0.20      0.00  pidstat
08:01:32 PM     0     16636      1.00      0.00  aliyun_assist_u
08:01:32 PM     0     20846      0.40      0.00  aliyun-service
08:01:32 PM     0     22337      0.20      0.00  sshd
08:01:32 PM     0     22362      0.20      0.00  sshd
08:01:32 PM     0     24080      0.20      0.00  AliYunDunUpdate
08:01:32 PM     0     24144     10.02      0.00  AliYunDun
08:01:32 PM     0     29618      0.40      0.00  sshd
08:01:32 PM     0     30492      0.20      0.00  kworker/u2:0
08:01:32 PM     0     31148      8.02      0.00  kworker/0:0
08:01:32 PM     0     31927     10.02      0.00  AliSecGuard
```

cswch，表示每秒自愿上下文切换（voluntary context switches）的次数。（所谓自愿上下文切换，是指进程无法获取所需资源，导致的上下文切换。比如说， I/O、内存等系统资源不足时，就会发生自愿上下文切换）

nvcswch  ，表示每秒非自愿上下文切换（non voluntary context switches）的次数。（非自愿上下文切换，则是指进程由于时间片已到等原因，被系统强制调度，进而发生的上下文切换。比如说，大量进程都在争抢 CPU 时，就容易发生非自愿上下文切换。）

### 上下文切换的案例

```
// 以10个线程运行5分钟的基准测试，模拟多线程切换的问题
[root@izbp1goz1ulmtus2vec80fz ~]# sysbench --threads=10 --max-time=300 threads run
WARNING: --max-time is deprecated, use --time instead
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 10
Initializing random number generator from current time


Initializing worker threads...

Threads started!
```

```
//每隔1秒输出1组数据，可以查看cpu上下文切换和中断数
[root@izbp1goz1ulmtus2vec80fz tools]# vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
16  0      0 1293500  21368 361056    0    0     1     3    0    1  0  0 99  0  0
 0  0      0 1293532  21368 361060    0    0     0     0  505  939  1  1 98  0  0
 0  0      0 1293532  21368 361060    0    0     0     0  557  950  0  1 99  0  0
 0  0      0 1293532  21368 361060    0    0     0     0  577  977  1  0 99  0  0
 8  0      0 1287900  21368 364560    0    0  3448     0  879 605378  3 38 56  3  0
 9  0      0 1287776  21368 364552    0    0     0     0 1116 1402511  9 91  0  0  0
 8  0      0 1287652  21368 364552    0    0     0     0 1338 1619876  9 91  0  0  0
 9  0      0 1287652  21368 364556    0    0     0     0 1197 1372570  8 92  0  0  0
 8  0      0 1287652  21368 364556    0    0     0     0 1248 1572861  7 93  0  0  0
 3  0      0 1287304  21368 364556    0    0     0     0 1029 1262244  7 93  0  0  0
 8  0      0 1287304  21376 364580    0    0     0    20 1256 1544621  9 91  0  0  0
 7  0      0 1287304  21376 364588    0    0     0     4  993 1195727  8 92  0  0  0
 7  0      0 1287304  21376 364588    0    0     0     0 1244 1623519  8 92  0  0  0
 8  0      0 1287304  21376 364556    0    0     0     0 1074 1401228  9 91  0  0  0
 8  0      0 1287304  21376 364556    0    0     0     0 1250 1661358  9 91  0  0  0
 9  0      0 1287304  21376 364556    0    0     0     0 1266 1548634  8 92  0  0  0
 7  0      0 1287304  21380 364564    0    0     0     4 1246 1546345  9 91  0  0  0
 9  0      0 1287304  21380 364568    0    0     0     0 1114 1376126 10 90  0  0  0
```

us（user）和 sy（system）列：这两列的 CPU 使用率加起来上升到了 100%，大部分是系统内核占用。

```
//  -w参数表示输出进程切换指标，而-u参数则表示输出CPU使用指标
[root@izbp1goz1ulmtus2vec80fz ~]# pidstat -w -u 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

08:30:23 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
08:30:24 PM     0     17370   10.11  100.00    0.00  100.00     0  sysbench
08:30:24 PM     0     24080    0.00    1.12    0.00    1.12     0  AliYunDunUpdate
08:30:24 PM     0     24144    1.12    0.00    0.00    1.12     0  AliYunDun

08:30:23 PM   UID       PID   cswch/s nvcswch/s  Command
08:30:24 PM     0         1      1.12      0.00  systemd
08:30:24 PM     0         3      2.25      0.00  ksoftirqd/0
08:30:24 PM     0         9     57.30      0.00  rcu_sched
08:30:24 PM     0      2117     11.24      0.00  redis-server
08:30:24 PM     0      2267     11.24      0.00  redis-server
08:30:24 PM     0     16299      1.12      0.00  vmstat
08:30:24 PM     0     16636      1.12      0.00  aliyun_assist_u
08:30:24 PM     0     17477      1.12      1.12  pidstat
08:30:24 PM     0     22362      1.12      0.00  sshd
08:30:24 PM     0     24144     10.11      0.00  AliYunDun
08:30:24 PM     0     31148      8.99      0.00  kworker/0:0
08:30:24 PM     0     31927     10.11      0.00  AliSecGuard
```

pidstat 的输出你可以发现，CPU 使用率的升高果然是 sysbench 导致的（使用率达到100%）

```
//-wt 参数表示输出线程的上下文切换指标
[root@izbp1goz1ulmtus2vec80fz ~]#  pidstat -wt 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz) 	05/28/2020 	_x86_64_	(1 CPU)

08:37:01 PM   UID      TGID       TID   cswch/s nvcswch/s  Command
08:37:02 PM     0         1         -      0.97      0.00  systemd
08:37:02 PM     0         -         1      0.97      0.00  |__systemd
08:37:02 PM     0         3         -      2.91      0.00  ksoftirqd/0
08:37:02 PM     0         -         3      2.91      0.00  |__ksoftirqd/0
08:37:02 PM     0         9         -     54.37      0.00  rcu_sched
08:37:02 PM     0         -         9     54.37      0.00  |__rcu_sched
08:37:02 PM     0        10         -      0.97      0.00  watchdog/0
08:37:02 PM     0         -        10      0.97      0.00  |__watchdog/0
08:37:02 PM     0        27         -      0.97      0.00  khugepaged
08:37:02 PM     0         -        27      0.97      0.00  |__khugepaged
08:37:02 PM     0      2117         -      9.71      0.00  redis-server
08:37:02 PM     0         -      2117      9.71      0.00  |__redis-server
08:37:02 PM     0      2267         -      9.71      0.00  redis-server
08:37:02 PM     0         -      2267      9.71      0.00  |__redis-server
08:37:02 PM     0     16299         -      0.97      0.00  vmstat
08:37:02 PM     0         -     16299      0.97      0.00  |__vmstat
08:37:02 PM     0     16636         -      0.97      0.00  aliyun_assist_u
08:37:02 PM     0         -     16636      0.97      0.00  |__aliyun_assist_u
08:37:02 PM     0         -     17807  36494.17 102912.62  |__sysbench
08:37:02 PM     0         -     17808  18465.05 123849.51  |__sysbench
08:37:02 PM     0         -     17809  22156.31 114126.21  |__sysbench
08:37:02 PM     0         -     17810  19675.73 129942.72  |__sysbench
08:37:02 PM     0         -     17811  22506.80 120136.89  |__sysbench
08:37:02 PM     0         -     17812  13093.20 132960.19  |__sysbench
08:37:02 PM     0         -     17813  23239.81 116176.70  |__sysbench
08:37:02 PM     0         -     17814  18564.08 123300.97  |__sysbench
08:37:02 PM     0         -     17815  31272.82 110352.43  |__sysbench
08:37:02 PM     0         -     17816  23869.90 120272.82  |__sysbench
```

```
// /proc/interrupts提供了一个只读的中断使用情况。
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/interrupts 
           CPU0       
  0:        140   IO-APIC-edge      timer
  1:       1544   IO-APIC-edge      i8042
  4:         58   IO-APIC-edge      serial
  6:          3   IO-APIC-edge      floppy
  8:          0   IO-APIC-edge      rtc0
  9:          0   IO-APIC-fasteoi   acpi
 11:        167   IO-APIC-fasteoi   uhci_hcd:usb1, virtio3
 12:       2160   IO-APIC-edge      i8042
 14:          0   IO-APIC-edge      ata_piix
 15:          0   IO-APIC-edge      ata_piix
 24:          0   PCI-MSI-edge      virtio2-config
 25:    7308756   PCI-MSI-edge      virtio2-req.0
 26:          8   PCI-MSI-edge      virtio0-config
 27:   41802278   PCI-MSI-edge      virtio0-input.0
 28:          1   PCI-MSI-edge      virtio0-output.0
 29:          0   PCI-MSI-edge      virtio1-config
 30:        290   PCI-MSI-edge      virtio1-virtqueues
NMI:          0   Non-maskable interrupts
LOC: 2262391990   Local timer interrupts
SPU:          0   Spurious interrupts
PMI:          0   Performance monitoring interrupts
IWI:  756043293   IRQ work interrupts
RTR:          0   APIC ICR read retries
RES:          0   Rescheduling interrupts
CAL:          0   Function call interrupts
TLB:          0   TLB shootdowns
TRM:          0   Thermal event interrupts
THR:          0   Threshold APIC interrupts
DFR:          0   Deferred Error APIC interrupts
MCE:          0   Machine check exceptions
MCP:     224992   Machine check polls
ERR:          0
MIS:          0
PIN:          0   Posted-interrupt notification event
PIW:          0   Posted-interrupt wakeup event
```

测试结果跟作者不太一致，观察下来是这个LOC: 2262391990   Local timer interrupts最多，而不是Rescheduling interrupts。

自愿上下文切换变多了，说明进程都在等待资源，有可能发生了 I/O 等其他问题；

非自愿上下文切换变多了，说明进程都在被强制调度，也就是都在争抢 CPU，说明 CPU 的确成了瓶颈；

/proc/interrupts 文件来分析具体的中断类型。

## 总结

可以借助 vmstat  、  pidstat 和 /proc/interrupts来查看上下文切换次数过多的问题。