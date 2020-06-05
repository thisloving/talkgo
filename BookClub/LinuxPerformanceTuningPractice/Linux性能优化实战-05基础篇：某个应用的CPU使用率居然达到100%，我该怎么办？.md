## Top 各项含义

+ uptime 的输出项
+ user（通常缩写为 us），代表用户态 CPU 时间。
+ nice（通常缩写为 ni），代表低优先级用户态 CPU 时间，也就是进程的 nice 值被调整为 1-19 之间时的 CPU 时间。
+ system（通常缩写为 sys），代表内核态 CPU 时间。
+ idle（通常缩写为 id），代表空闲时间。
+ iowait（通常缩写为 wa），代表等待 I/O 的 CPU 时间。
+ irq（通常缩写为 hi），代表处理硬中断的 CPU 时间。
+ softirq（通常缩写为 si），代表处理软中断的 CPU 时间。
+ steal（通常缩写为 st），代表当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间。

进入top界面后：

+ M —根据驻留内存大小进行排序
+ P —根据CPU使用百分比大小进行排序
+ T —根据时间/累计时间进行排序
+ c —切换显示命令名称和完整命令行
+ t —切换显示进程和CPU信息
+ m —切换显示内存信息
+ l —切换显示平均负载和启动时间信息
+ o —改变显示项目的顺序
+ f —从当前显示中添加或删除项目
+ S —切换到累计模式
+ s —改变两次刷新之间的延迟时间。系统将提示用户输入新的时间，单位为s。如果有小数，就换算成ms。输入0值则系统将不断刷新，默认值是5s；如果值设的很小，不但看不清结果，同时还会是系统负载大大增加
+ q —退出top程序
+ i —忽略闲置和僵尸进程。这是一个开关式的命令
+ k —终止一个进程，系统将提示用户输入需要终止的进程PID，以及需要发送给该进程什么样的信号。一般终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束进程；默认的信号是15；但是在安全模式中此命令被屏蔽

## CPU使用率

CPU 使用率，就是除了空闲时间外的其他时间占总 CPU 时间的百分比。

```
cpu使用率 = 1 - (空闲时间/总cpu时间)
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/stat 
cpu  24018022 4032 19833085 6706912426 650486 0 20737 0 0 0
cpu0 24018022 4032 19833085 6706912426 650486 0 20737 0 0 0
intr 3313549165 140 1544 0 0 58 0 3 0 0 0 0 167 2160 0 0 0 0 0 0 0 0 0 0 0 0 7375078 8 41991912 1 0 290 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
ctxt 58991682334
btime 1523172454
processes 21501765
procs_running 4
procs_blocked 0
softirq 5419156130 41 2912707062 44 89526830 0 0 1817 0 0 2416920336
```

更有价值的是平均cpu使用率

```
平均cpu使用率 = 1 - （空间时间new - 空闲时间old）/(总cpu时间new - 总cpu时间old)
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/2117/stat
2117 (redis-server) S 1 2117 2117 0 -1 4202816 1713 1312 26 0 400571 545134 0 0 20 0 4 0 5105613660 150896640 547 18446744073709551615 4194304 5501332 140723379774240 140723379773776 139668859167875 0 0 4097 17610 18446744073709551615 0 0 17 0 0 0 432 0 0 7601536 7622193 10743808 140723379779582 140723379779616 140723379779616 140723379781596 0
```

## 怎么查看cpu使用率

```
[root@izbp1goz1ulmtus2vec80fz ~]# top
top - 15:49:44 up 784 days, 22 min,  4 users,  load average: 0.11, 0.05, 0.05
Tasks:  86 total,   1 running,  85 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  6.2 sy,  0.0 ni, 93.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883492 total,   859556 free,   204728 used,   819208 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1491500 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                              
    1 root      20   0   51732   3040   1660 S  0.0  0.2  58:13.57 systemd                                                                                              
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.09 kthreadd                                                                                             
    3 root      20   0       0      0      0 S  0.0  0.0   2:32.43 ksoftirqd/0  
```

top 并没有细分进程的用户态 CPU 和内核态 CPU，查看每个进程的详细情况，可以使用pidstat 命令。

```
[root@izbp1goz1ulmtus2vec80fz ~]# pidstat 1 5
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       05/31/2020      _x86_64_        (1 CPU)
03:51:31 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
03:51:32 PM     0     24144    0.00    1.00    0.00    1.00     0  AliYunDun
03:51:33 PM     0     24144    1.01    0.00    0.00    1.01     0  AliYunDun
```

+ %usr，用户态 CPU 使用率
+ %system，内核态 CPU 使用率
+ %guest ，运行虚拟机 CPU 使用率
+ %wait，等待 CPU 使用率（测试中没看见）

## CPU 使用率过高怎么办？

perf 是 Linux 2.6.31 以后内置的性能分析工具。它以性能事件采样为基础，不仅可以分析系统的各种事件和内核性能，还可以用来分析指定应用程序的性能问题。

```
[root@izbp1goz1ulmtus2vec80fz ~]# perf top
Samples: 1K of event 'cpu-clock', 4000 Hz, Event count (approx.): 409843750 lost: 0/0 drop: 0/0                                                                         
Overhead  Shared Object            Symbol                                                                                                                               
  10.89%  perf                     [.] __symbols__insert
  10.41%  perf                     [.] rb_next
   3.37%  [kernel]                 [k] _raw_spin_unlock_irqrestore
   3.36%  [kernel]                 [k] kallsyms_expand_symbol.constprop.1
   3.15%  perf                     [.] 0x00000000001be397
   3.09%  [kernel]                 [k] finish_task_switch
```

持久化perf record 生成perf.data，perf report 读取相关信息

## 案例

启动一个web服务器，用ab测试。（注：没有选择示例中的php+nginx，采用的go web）

```
package main

import (
	"math"
	"net/http"
)

type helloHandler struct{}

func loop() {
	x := 0
	for i := 0; i < 1000000; i++ {
		x += int(math.Sqrt(float64(x)))
	}
}

func (h *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	loop()
	w.Write([]byte("Hello, world!"))
}

func main() {
	http.Handle("/", &helloHandler{})
	http.ListenAndServe(":12345", nil)
}
```

```
// 运行结果
[root@izbp1goz1ulmtus2vec80fz Experiment]# curl http://localhost:12345
Hello, world!
```

下面用ab测试

```
// ab（apache bench）是一个常用的 HTTP 服务性能测试工具，类似的还有Locust
ab -c 1000 -n 10000 http://localhost:12345/
```

```
// top或者htop显示
[root@izbp1goz1ulmtus2vec80fz ~]# top
top - 16:28:16 up 784 days,  1:00,  6 users,  load average: 0.83, 0.30, 0.18
Tasks:  92 total,   1 running,  91 sleeping,   0 stopped,   0 zombie
%Cpu(s): 94.7 us,  3.7 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  1.7 si,  0.0 st
KiB Mem :  1883492 total,   514232 free,   250328 used,  1118932 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1425380 avail Mem 

PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                 

25133 root      20   0  138800  16604   2420 S 91.0  0.9   0:14.77 web 
```

从上面实现可以看到，top或者htop可以清楚的看见是web这个进程导致cpu占用过高，接下来想知道是哪个函数导致cpu占用过高？

下面使用perf来实际演练一下：

```
// 25112是web线程id
perf top -g -p 25112
```

运行结果展示：

```
Samples: 18K of event 'cpu-clock', 4000 Hz, Event count (approx.): 3540281967 lost: 0/0 drop: 0/0                                                                       
  Children      Self  Shared Object     Symbol                                                                                                                          
+   86.80%     0.02%  web               [.] net/http.serverHandler.ServeHTTP
+   86.55%    85.72%  web               [.] main.loop
+   74.78%     0.12%  web               [.] net/http.(*conn).serve
+   67.51%     0.02%  web               [.] net/http.(*ServeMux).ServeHTTP
+   38.81%     0.01%  web               [.] runtime.goexit
+    3.71%     0.05%  web               [.] syscall.Syscall
+    3.21%     0.01%  [kernel]          [k] system_call_fastpath
+    2.37%     0.05%  web               [.] net/http.(*conn).readRequest
+    2.29%     0.08%  [kernel]          [k] tcp_transmit_skb
+    2.24%     0.00%  [kernel]          [k] __tcp_push_pending_frames
+    2.24%     0.05%  [kernel]          [k] tcp_write_xmit
+    2.16%     0.00%  web               [.] syscall.Write
+    2.14%     0.04%  [kernel]          [k] __do_softirq
+    2.12%     0.07%  [kernel]          [k] ip_queue_xmit
+    2.07%     0.00%  [kernel]          [k] vfs_write
+    2.02%     0.06%  [kernel]          [k] tcp_sendmsg
+    1.79%     0.03%  web               [.] net/http.readRequest
+    1.76%     0.01%  web               [.] net.(*netFD).Write
+    1.74%     0.01%  web               [.] internal/poll.(*FD).Write
+    1.54%     0.02%  [kernel]          [k] ip_output
+    1.53%     0.02%  [kernel]          [k] net_rx_action
+    1.46%     0.02%  [kernel]          [k] __netif_receive_skb_core
+    1.41%     0.14%  [kernel]          [k] __local_bh_enable_ip
+    1.33%     0.01%  web               [.] net/http.(*response).finishRequest
+    1.23%     0.01%  web               [.] net.(*netFD).Close
     1.19%     0.01%  web               [.] net.(*netFD).accept
+    1.17%     0.06%  [kernel]          [k] tcp_v4_rcv
+    1.17%     0.02%  [kernel]          [k] process_backlog
+    1.16%     0.01%  [kernel]          [k] ip_finish_output
+    1.15%     0.01%  web               [.] internal/poll.(*FD).Close
+    1.14%     0.01%  web               [.] bufio.(*Writer).Flush
+    1.14%     0.02%  [kernel]          [k] ip_rcv
     1.07%     0.04%  web               [.] net.(*TCPAddr).String
     1.06%     0.02%  web               [.] runtime.systemstack
     1.05%     0.02%  [kernel]          [k] tcp_v4_do_rcv
     0.98%     0.20%  web               [.] runtime.gentraceback
     0.96%     0.02%  [kernel]          [k] task_work_run
```

### 很清晰的显示是main.loop耗时过长。

## 总结

碰到 CPU 使用率升高的问题，你可以借助 top、pidstat 等工具，确认引发 CPU 性能问题的来源；再使用 perf 等工具，排查出引起性能问题的具体函数。