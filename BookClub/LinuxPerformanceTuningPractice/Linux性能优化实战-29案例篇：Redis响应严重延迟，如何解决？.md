## 案例分析

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# docker run --name=redis -itd -p 10000:80 feisky/redis-server
Unable to find image 'feisky/redis-server:latest' locally
latest: Pulling from feisky/redis-server
cd784148e348: Pull complete 
48d4c7155ddc: Pull complete 
6d908603dbe8: Pull complete 
0b981e82e1e2: Pull complete 
7074f4a1fd03: Pull complete 
447ac2b250dc: Pull complete 
b6d44ce71e94: Pull complete 
Digest: sha256:a69d39256eb970ab0d87a70d53fa2666d0c32cbf68fb316ef016efd513806146
Status: Downloaded newer image for feisky/redis-server:latest
761672b42e18f429695ba764cff28f75b69c4fce7cd30f650c9884f9e30040f0
```

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# docker run --name=app --network=container:redis -itd feisky/redis-app
Unable to find image 'feisky/redis-app:latest' locally
latest: Pulling from feisky/redis-app
54f7e8ac135a: Pull complete 
d6341e30912f: Pull complete 
087a57faf949: Pull complete 
5d71636fb824: Pull complete 
0c1db9598990: Pull complete 
2eeb5ce9b924: Pull complete 
d3029c597b32: Pull complete 
265a9c957eba: Pull complete 
3bb7ae9463c5: Pull complete 
b3198935e7ab: Pull complete 
ca3ab58d03d9: Pull complete 
Digest: sha256:ac281cbb37e35eccb880a58b75f244ef19a9a8704a43ae274c12e6576ed6082b
Status: Downloaded newer image for feisky/redis-app:latest
```

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED              STATUS              PORTS                             NAMES
a7fbf34019f9        feisky/redis-app      "python /app.py"         About a minute ago   Up 59 seconds                                         app
761672b42e18        feisky/redis-server   "docker-entrypoint.s…"   14 minutes ago       Up 14 minutes       6379/tcp, 0.0.0.0:10000->80/tcp   redis
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# curl http://localhost:10000/   
hello redis
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# curl http://localhost:10000/init/5000   
{"elapsed_seconds":15.537309885025024,"keys_initialized":5000}
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# curl http://localhost:10000/get_cache   
{"count":1625,"data":["ebe20d1a-b3dd-11ea-b8dd-0242ac110006","ef45643e-b3dd-11ea-b8dd-0242ac110006","ea637e88-b3dd-11ea-b8dd-0242ac110006","ee3c474c-b3dd-11ea-b8dd-0242ac110006","ef9cea38-b3dd-11ea-b8dd-0242ac110006","e8c27bec-b3dd-11ea-b8dd-0242ac110006"
```

```
while true; do curl http://192.168.0.10:10000/get_cache; done
```

运行后界面一直在输出

```
Tasks:  96 total,   2 running,  94 sleeping,   0 stopped,   0 zombie
%Cpu(s):  9.1 us,  5.1 sy,  0.0 ni,  0.0 id, 84.8 wa,  0.0 hi,  1.0 si,  0.0 st
KiB Mem :  1883492 total,    72724 free,   273776 used,  1536992 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used.  1380828 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                              
 9772 root      20   0  193248  23968   5148 S  9.0  1.3   0:07.96 python                                                                                               
 8927 100       20   0   28340   3028   1136 D  4.3  0.2   0:04.81 redis-server       
```

 iowait 比较高，已经达到了 84%；而各个进程的 CPU 使用率都不太高，最高的 python 和 redis-server ，

综合 top 的信息，最有嫌疑的就是 iowait。所以，接下来还是要继续分析，是不是 I/O 问题。

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# iostat -d -x 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       06/22/2020      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.08    0.01    0.21     1.27     5.25    59.96     0.00   22.18   14.31   22.62   0.92   0.02

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00   396.97    0.00 1135.35     0.00  6153.54    10.84     0.84    0.74    0.00    0.74   0.74  84.44
```

跟作者并不一样，util比较高。

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# pidstat -d 1
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       06/22/2020      _x86_64_        (1 CPU)

12:46:10 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
12:46:11 AM   100      8927      0.00    992.00      0.00  redis-server

12:46:11 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
12:46:12 AM     0       255      0.00      4.04      0.00  jbd2/vda1-8
12:46:12 AM   100      8927      0.00    933.33      0.00  redis-server

12:46:12 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
12:46:13 AM   100      8927      0.00   1204.08      0.00  redis-server

12:46:13 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
12:46:14 AM   100      8927      0.00    893.88      0.00  redis-server
```

I/O 最多的进程是 PID 为 8927的 redis-server，并且它也刚好是在写磁盘。这说明，确实是 redis-server 在进行磁盘写。

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# strace -f -T -tt -p 8927
[pid  8927] 00:47:16.967595 write(8, "$6\r\nnormal\r\n", 12) = 12 <0.000065>
[pid  8927] 00:47:16.967682 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 24, NULL, 8) = 1 <0.000011>
[pid  8927] 00:47:16.967715 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:ea74c864-"..., 16384) = 61 <0.000011>
[pid  8927] 00:47:16.967749 read(3, 0x7fff442f0157, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000010>
[pid  8927] 00:47:16.967778 write(8, "$6\r\nnormal\r\n", 12) = 12 <0.000063>
[pid  8927] 00:47:16.967863 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 24, NULL, 8) = 1 <0.000011>
[pid  8927] 00:47:16.967894 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:e9322e4c-"..., 16384) = 61 <0.000011>
[pid  8927] 00:47:16.967933 read(3, 0x7fff442f0157, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000010>
[pid  8927] 00:47:16.967963 write(8, "$3\r\nbad\r\n", 9) = 9 <0.000064>
[pid  8927] 00:47:16.968055 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 23, NULL, 8) = 1 <0.000021>
[pid  8927] 00:47:16.968098 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:f08bdd78-"..., 16384) = 61 <0.000011>
[pid  8927] 00:47:16.968133 read(3, 0x7fff442f0157, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000011>
[pid  8927] 00:47:16.968163 write(8, "$4\r\ngood\r\n", 10) = 10 <0.000074>
[pid  8927] 00:47:16.968259 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 23, NULL, 8) = 1 <0.000011>
[pid  8927] 00:47:16.968291 read(8, "*3\r\n$4\r\nSADD\r\n$4\r\ngood\r\n$36\r\nf08"..., 16384) = 67 <0.000010>
[pid  8927] 00:47:16.968329 read(3, 0x7fff442f0157, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000010>
[pid  8927] 00:47:16.968363 write(7, "*3\r\n$4\r\nSADD\r\n$4\r\ngood\r\n$36\r\nf08"..., 67) = 67 <0.000022>
[pid  8927] 00:47:16.968407 fdatasync(7) = 0 <0.002176>
[pid  8927] 00:47:16.970609 write(8, ":1\r\n", 4) = 4 <0.000098>
[pid  8927] 00:47:16.970734 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 21, NULL, 8) = 1 <0.000012>
[pid  8927] 00:47:16.970770 read(8, "*4\r\n$4\r\nSCAN\r\n$4\r\n4240\r\n$5\r\nMATC"..., 16384) = 47 <0.000010>
[pid  8927] 00:47:16.970818 read(3, 0x7fff442f0157, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000010>
[pid  8927] 00:47:16.970849 write(8, "*2\r\n$4\r\n1680\r\n*10\r\n$41\r\nuuid:ebd"..., 499) = 499 <0.000284>
[pid  8927] 00:47:16.971161 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 20, NULL, 8) = 1 <0.000011>
[pid  8927] 00:47:16.971198 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:ebdbd5da-"..., 16384) = 61 <0.000012>
[pid  8927] 00:47:16.971235 read(3, 0x7fff442f0157, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000010>
[pid  8927] 00:47:16.971266 write(8, "$4\r\ngood\r\n", 10) = 10 <0.000089>
[pid  8927] 00:47:16.971382 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 20, NULL, 8) = 1 <0.000012>
[pid  8927] 00:47:16.971418 read(8, "*3\r\n$4\r\nSADD\r\n$4\r\ngood\r\n$36\r\nebd"..., 16384) = 67 <0.000011>
```

从系统调用来看， epoll_pwait、read、write、fdatasync 这些系统调用都比较频繁。那么，刚才观察到的写磁盘，应该就是 write 或者 fdatasync 导致的了。

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]# lsof -p 8927  
redis-ser 8927      100    8u     sock    0,7      0t0 26383922 protocol: TCP
redis-ser 8927      100    5u  a_inode    0,9        0     4852 [eventpoll]
redis-ser 8927      100    3r     FIFO    0,8      0t0 26381551 pipe
redis-ser 8927      100    4w     FIFO    0,8      0t0 26381551 pipe
redis-ser 8927      100    7w      REG  253,1  8086102   274643 /data/appendonly.aof
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# docker exec -it redis redis-cli config get 'append*'
1) "appendfsync"
2) "always"
3) "appendonly"
4) "yes"
```

```
[root@izbp1goz1ulmtus2vec80fz linux-perf-examples]#  strace -f -p 8927 -T -tt -e fdatasync | more
strace: Process 8927 attached with 4 threads
[pid  8927] 00:52:08.371554 fdatasync(7) = 0 <0.002388>
[pid  8927] 00:52:08.374356 fdatasync(7) = 0 <0.002808>
[pid  8927] 00:52:08.377719 fdatasync(7) = 0 <0.002276>
[pid  8927] 00:52:08.380640 fdatasync(7) = 0 <0.004670>
[pid  8927] 00:52:08.385984 fdatasync(7) = 0 <0.002585>
[pid  8927] 00:52:08.389134 fdatasync(7) = 0 <0.002684>
[pid  8927] 00:52:08.392878 fdatasync(7) = 0 <0.002704>
[pid  8927] 00:52:08.396341 fdatasync(7) = 0 <0.005106>
[pid  8927] 00:52:08.402062 fdatasync(7) = 0 <0.002704>
[pid  8927] 00:52:08.406088 fdatasync(7) = 0 <0.004247>
[pid  8927] 00:52:08.411617 fdatasync(7) = 0 <0.002551>
[pid  8927] 00:52:08.414503 fdatasync(7) = 0 <0.002929>
[pid  8927] 00:52:08.418288 fdatasync(7) = 0 <0.003148>
```

Redis 配置的 appendfsync 是 always，这就导致 Redis 每次的写操作，都会触发 fdatasync 系统调用。今天的案例，没必要用这么高频的同步写，使用默认的 1s 时间间隔，就足够了。