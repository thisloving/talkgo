## 案例

### 准备

+ sar 是一个系统活动报告工具，既可以实时查看系统的当前活动，又可以配置保存和报告历史统计数据。
+ hping3 是一个可以构造 TCP/IP 协议数据包的工具，可以对系统进行安全审计、防火墙测试等。
+ tcpdump 是一个常用的网络抓包工具，常用来分析各种网络问题。

按照作者的思路做了一次，2台阿里云，效果并不明显

第一台机器47.X.X.3上 运行指令

```
docker run -itd --name=nginx -p 80:80 nginx
```

第二台机器47.X.X.97上运行指令

```
[root@izbp1goz1ulmtus2vec80fz ~]# curl http://47.X.X.3/  
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

接着，在第二个终端，运行 hping3 命令，来模拟 Nginx 的客户端请求：

```
hping3 -S -p 80 -i u100 47.X.X.3
```

在第一机器上查看

```
[root@leving-hk ~]# top
top - 17:51:57 up 241 days,  2:50,  6 users,  load average: 0.00, 0.02, 0.05
Tasks:  88 total,   2 running,  86 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.0 sy,  0.0 ni, 93.3 id,  0.0 wa,  0.0 hi,  6.4 si,  0.0 st
KiB Mem :  1014908 total,    74812 free,   184284 used,   755812 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   595736 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                            
    3 root      20   0       0      0      0 S  0.3  0.0   1:25.13 ksoftirqd/0                                                        
    9 root      20   0       0      0      0 R  0.3  0.0   1:43.73 rcu_sched                                                          
 6278 root      20   0  303616  11524   5488 S  0.3  1.1   0:00.81 docker-containe                                                    
 7206 root      20   0  161896   2272   1612 R  0.3  0.2   0:00.14 top                                                                
    1 root      20   0  125472   3100   1744 S  0.0  0.3  20:40.31 systemd                                                            
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.31 kthreadd                                                           
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H                                                       
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0                                                        
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh                                                             
   10 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 lru-add-drain                                                      
   11 root      rt   0       0      0      0 S  0.0  0.0   1:57.64 watchdog/0                                                         
   13 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kdevtmpfs  
```

因为这条机器本身在香港，所以访问上有点卡。top看到的信息没有作者那么明显。

还是在第一个终端，运行下面的命令

```
[root@leving-hk ~]# watch -d cat /proc/softirqs
Every 2.0s: cat /proc/softirqs                                                                                Sun Jun  7 17:38:24 2020

                    CPU0
          HI:          9
       TIMER:  216219622
      NET_TX:          2
      NET_RX:   57825006
       BLOCK:          0
BLOCK_IOPOLL:          0
     TASKLET:         89
       SCHED:          0
     HRTIMER:          0
         RCU:   74975224
```

第一个终端中运行 sar 命令，并添加 -n DEV 参数显示网络收发的报告：

```
#sar -n DEV 1
05:51:46 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:51:47 PM veth918a81c   2393.94   2632.32    135.59    138.81      0.00      0.00      0.00
05:51:47 PM      eth0   2637.37   2395.96    154.53    137.63      0.00      0.00      0.00
05:51:47 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:51:47 PM   docker0   2393.94   2632.32    102.86    138.81      0.00      0.00      0.00
```

这个实验做的并不是太成功，主要熟悉网络指令。

因为涉及到实际ip，tcpdump信息就不贴了。