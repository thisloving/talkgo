## 案例分析

上一节能直接看到cpu使用率，这一节看另外一个例子。发现系统的 CPU 使用率很高的时候，不一定能找到相对应的高 CPU 使用率的进程。

```
//example
https://github.com/feiskyer/linux-perf-examples/tree/master/nginx-short-process
```

```
[root@izbp1goz1ulmtus2vec80fz Experiment]# ab -c 100 -n 1000 http://localhost:10000/   
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        nginx/1.15.4
Server Hostname:        localhost
Server Port:            10000

Document Path:          /
Document Length:        9 bytes

Concurrency Level:      100
Time taken for tests:   17.218 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      172000 bytes
HTML transferred:       9000 bytes
Requests per second:    58.08 [#/sec] (mean)
Time per request:       1721.754 [ms] (mean)
Time per request:       17.218 [ms] (mean, across all concurrent requests)
Transfer rate:          9.76 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.4      0       2
Processing:    21 1646 340.8   1724    2242
Waiting:       19 1646 340.8   1724    2242
Total:         21 1646 340.5   1724    2242

Percentage of the requests served within a certain time (ms)
  50%   1724
  66%   1759
  75%   1778
  80%   1841
  90%   1989
  95%   2034
  98%   2054
  99%   2068
 100%   2242 (longest request)
```

```
//top查看到的信息
[root@izbp1goz1ulmtus2vec80fz Experiment]# top
top - 23:01:44 up 784 days,  7:34, 11 users,  load average: 2.22, 0.66, 0.40
Tasks: 122 total,   9 running, 113 sleeping,   0 stopped,   0 zombie
%Cpu(s): 78.4 us, 21.2 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.4 si,  0.0 st
KiB Mem :  1883492 total,    76012 free,  1286776 used,   520704 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   390036 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                              
24144 root      10 -10  158932  41476   4948 S  6.3  2.2 336:02.73 AliYunDun                                                                                            
  564 root      20   0 1322524   1.0g   2300 S  2.7 54.6   1:36.10 perf                                                                                                 
29243 root      20   0  282524   8240   1756 S  1.7  0.4   0:01.06 docker-proxy                                                                                         
 7806 bin       20   0  336684   7688   1904 S  1.3  0.4   0:00.11 php-fpm                                                                                              
29301 101       20   0   34016   2984    776 S  1.3  0.2   0:00.93 nginx                                                                                                
 7798 root      20   0   61668   3756   2436 R  1.0  0.2   0:00.07 ab                                                                                                   
 7813 bin       20   0  336684   7688   1904 S  1.0  0.4   0:00.10 php-fpm                                                                                              
 7815 bin       20   0  336684   7684   1900 S  1.0  0.4   0:00.10 php-fpm                                                                                              
 7799 bin       20   0  336684   7684   1900 S  0.7  0.4   0:00.10 php-fpm                                                                                              
 7825 bin       20   0  336684   7684   1900 S  0.7  0.4   0:00.10 php-fpm     
```

观察 top 输出的进程列表可以发现，CPU 使用率最高的进程也只不过才 6.3%，看起来并不高。再看系统 CPU 使用率（ %Cpu ）这一行，你会发现，系统的整体 CPU 使用率是比较高的：用户 CPU 使用率（us）已经到了 78.4 %，系统 CPU 为 21.2%，而空闲 CPU （id）则是 0.0%。

```
// 观察pidstat的输出
[root@izbp1goz1ulmtus2vec80fz ~]# pidstat 1
Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0         9    0.00    0.09    0.00    0.09     -  rcu_sched
Average:        0        25    0.00    0.04    0.00    0.04     -  kswapd0
Average:        0       564    2.26    0.22    0.00    2.48     -  perf
Average:        0       857    0.04    0.04    0.00    0.09     -  top
Average:        0      2117    0.04    0.00    0.00    0.04     -  redis-server
Average:        0      2267    0.04    0.04    0.00    0.09     -  redis-server
Average:        0      5853    0.00    0.04    0.00    0.04     -  kworker/0:1
Average:        0     10937    0.13    0.30    0.00    0.43     -  pidstat
Average:        0     10943    0.09    0.35    0.00    0.43     -  ab
Average:        1     10944    0.17    0.61    0.00    0.78     -  php-fpm
Average:        1     10951    0.17    0.61    0.00    0.78     -  php-fpm
Average:        1     10959    0.13    0.57    0.00    0.70     -  php-fpm
Average:        1     10962    0.17    0.57    0.00    0.74     -  php-fpm
Average:        1     10973    0.13    0.57    0.00    0.70     -  php-fpm
Average:        0     16636    0.04    0.00    0.00    0.04     -  aliyun_assist_u
Average:        0     20846    0.04    0.00    0.00    0.04     -  aliyun-service
Average:        0     24144    3.87    1.78    0.00    5.65     -  AliYunDun
Average:        0     29243    0.43    0.83    0.00    1.26     -  docker-proxy
Average:        0     29248    0.17    0.26    0.00    0.43     -  containerd-shim
Average:      101     29301    0.52    0.61    0.00    1.13     -  nginx
Average:        0     29621    0.00    0.04    0.00    0.04     -  containerd
Average:        0     29799    0.04    0.09    0.00    0.13     -  php-fpm
Average:        0     30128    0.26    0.17    0.00    0.43     -  dockerd
```

明明用户 CPU 使用率已经高达 100%，最高的是AliYunDun，cpu只有3.87。

```
// perf top 查看
[root@izbp1goz1ulmtus2vec80fz ~]# perf top
Samples: 80K of event 'cpu-clock', 4000 Hz, Event count (approx.): 11277468276 lost: 0/0 drop: 0/0                                                                      
Overhead  Shared Object             Symbol                                                                                                                              
   5.09%  libc-2.24.so              [.] 0x0000000000036387
   2.85%  stress                    [.] 0x0000000000002eff
   2.74%  stress                    [.] 0x0000000000002ee5
   2.73%  stress                    [.] 0x0000000000002f09
   2.70%  libc-2.24.so              [.] 0x0000000000036360
   2.67%  libc-2.24.so              [.] 0x00000000000363c8
   2.66%  libc-2.24.so              [.] 0x000000000003636b
   2.65%  stress                    [.] 0x0000000000000e80
   2.58%  libc-2.24.so              [.] 0x00000000000364f0
   2.57%  libc-2.24.so              [.] 0x0000000000036829
   2.56%  stress                    [.] 0x0000000000002ef3
   2.53%  libc-2.24.so              [.] 0x0000000000036820
   2.53%  libc-2.24.so              [.] 0x000000000003650c
   2.52%  libc-2.24.so              [.] 0x0000000000036522
   2.51%  libc-2.24.so              [.] 0x0000000000036538
   2.49%  libc-2.24.so              [.] 0x00000000000364fd
   2.47%  libc-2.24.so              [.] 0x00000000000363ea
   2.46%  libc-2.24.so              [.] 0x000000000003652e
   2.46%  libc-2.24.so              [.] 0x0000000000036518
   2.42%  stress                    [.] 0x0000000000002f25
   2.41%  stress                    [.] 0x0000000000002f18
   1.88%  [kernel]                  [k] __do_page_fault
   1.67%  [kernel]                  [k] mem_cgroup_charge_common
   1.29%  [kernel]                  [k] _raw_spin_unlock_irqrestore
   1.10%  [kernel]                  [k] unmap_page_range
   1.09%  stress                    [.] 0x0000000000002f29
   1.01%  stress                    [.] 0x0000000000002f1b
   0.89%  [kernel]                  [k] clear_page_c_e
   0.85%  libaegisProcMng.so        [.] CDockerProcessor::ProcExitEventCallBack
   0.82%  [kernel]                  [k] __raw_callee_save___pv_queued_spin_unlock
   0.78%  perf                      [.] __symbols__insert
   0.65%  [kernel]                  [k] get_page_from_freelist
   0.65%  stress                    [.] 0x0000000000002f0d
   0.59%  [kernel]                  [k] anon_vma_interval_tree_insert
   0.50%  perf                      [.] rb_next
   0.49%  [kernel]                  [k] copy_pte_range
   0.48%  libc-2.17.so              [.] _int_malloc
```

perf top 查看下来会发现许多stress程序

#### 原来是代码里调用了stress，stress是个短程序，基本不会在top上输出。

下面是是一个专为短时进程设计的工具 ,execsnoop 。它通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息，包括进程 PID、父进程 PID、命令行参数以及执行的结果。

```
[root@izbp1goz1ulmtus2vec80fz ~]# ./tools/perf-tools/execsnoop
 25778  25775 /usr/local/bin/stress -t 1 -d 1
 25780  17184 sh [?]
 25781  17217 sh [?]
 25783  25781 /usr/local/bin/stress -t 1 -d 1
 25782  25780 /usr/local/bin/stress -t 1 -d 1
 25785  17227 sh [?]
 25787  25785 /usr/local/bin/stress -t 1 -d 1
 25790  17207 sh [?]
 25789  17214 sh [?]
 25792  25790 /usr/local/bin/stress -t 1 -d 1
 25793  25789 /usr/local/bin/stress -t 1 -d 1
 25791  17227 sh [?]
 25797  25791 /usr/local/bin/stress -t 1 -d 1
 25794  17184 sh [?]
 25799  25794 /usr/local/bin/stress -t 1 -d 1
 25801  17217 sh [?]
 25803  25801 /usr/local/bin/stress -t 1 -d 1
 25802  17184 sh [?]
 25804  17227 sh [?]
 25805  17214 sh [?]
 25807  25802 /usr/local/bin/stress -t 1 -d 1
 25808  25804 /usr/local/bin/stress -t 1 -d 1
 25809  25805 /usr/local/bin/stress -t 1 -d 1
 25813  17207 sh [?]
 25814  25813 /usr/local/bin/stress -t 1 -d 1
 25816  17217 sh [?]
 25817  17184 sh [?]
 25818  25816 /usr/local/bin/stress -t 1 -d 1
 25819  25817 /usr/local/bin/stress -t 1 -d 1
 25822  17214 sh [?]
 25824  25822 /usr/local/bin/stress -t 1 -d 1
 25826  17217 sh [?]
 25823  17227 sh [?]
 25828  25826 /usr/local/bin/stress -t 1 -d 1
 25825  17184 sh [?]
 25829  25823 /usr/local/bin/stress -t 1 -d 1
 25831  25825 /usr/local/bin/stress -t 1 -d 1
```

## 总结

当发现系统的 CPU 使用率很高的时候，找不到相对应的高 CPU 使用率的进程，可以考虑用用短程序查看工具execsnoop 或者perf top查看。