## 案例分析

```
[root@izbp1goz1ulmtus2vec80fz ~]# docker run -v /tmp:/tmp --name=app -itd feisky/logapp 
bce56aee5deb0af229a9a4fd251e9329e4131db873b2787ee845f626c1511503
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# ps -ef | grep /app.py 
root      6648  6632 41 23:50 pts/0    00:00:38 python /app.py
root      6774 27822  0 23:51 pts/0    00:00:00 grep --color=auto /app.py
```

在终端中运行 top 命令，观察 CPU 和内存的使用情况：

```
[root@izbp1goz1ulmtus2vec80fz ~]# top
top - 23:54:37 up 805 days,  8:27,  4 users,  load average: 2.94, 1.90, 0.85
Tasks:  93 total,   2 running,  91 sleeping,   0 stopped,   0 zombie
%Cpu(s):  7.6 us, 40.2 sy,  0.0 ni,  0.0 id, 52.2 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883492 total,   519024 free,   744368 used,   620100 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used.   939260 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                              
 6648 root      20   0  656132 485628    956 R 42.9 25.8   1:46.39 python                                                                                               
   25 root      20   0       0      0      0 S  2.0  0.0   0:33.11 kswapd0  
```

 iowait比CPU 使用率（sys%）高，说明可能正在运行 I/O 密集型的进程。

```
[root@izbp1goz1ulmtus2vec80fz ~]# iostat -x -d 1 
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       06/21/2020      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.08    0.01    0.20     1.24     4.40    53.05     0.00   19.33   13.38   19.65   0.90   0.02

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     4.12    3.09   58.76    12.37 27439.18   887.60    46.35  193.28   15.33  202.65   6.42  39.69

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    5.49  361.54    26.37 172378.02   939.45   135.98  401.18   78.20  406.09   2.99 109.89
```

vda的%util I/O 使用率可能已经接近 I/O 饱和。

iowait 高说明正是磁盘 sda 的 I/O 瓶颈导致的。

接下来的重点就是分析 I/O 性能瓶颈的根源了。那要怎么知道，这些 I/O 请求相关的进程呢？

```
[root@izbp1goz1ulmtus2vec80fz ~]# pidstat -d 1 
Linux 3.10.0-693.2.2.el7.x86_64 (izbp1goz1ulmtus2vec80fz)       06/21/2020      _x86_64_        (1 CPU)
Average:      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
Average:        0       255      0.00   9470.61      0.00  jbd2/vda1-8
Average:        0      2267    662.26      0.00      0.00  redis-server
Average:        0      6648      0.00 106852.87      0.00  python
Average:        0     24144    461.22      0.00      0.00  AliYunDun
Average:        0     26863      4.17      0.00      0.00  kworker/u2:1
Average:        0     27819    405.57      0.00      0.00  sshd
```

从 pidstat 的输出，你可以发现，只有 python 进程的写比较大，很明显，正是 python 进程导致了 I/O 瓶颈。

查看python进程

````
[root@izbp1goz1ulmtus2vec80fz ~]# strace -p 6648
strace: Process 6648 attached
mremap(0x7fece8c71000, 393220096, 314576896, MREMAP_MAYMOVE) = 0x7fece8c71000
munmap(0x7fed00372000, 314576896)       = 0
lseek(3, 0, SEEK_END)                   = 943718535
lseek(3, 0, SEEK_CUR)                   = 943718535
munmap(0x7fece8c71000, 314576896)       = 0
close(3)                                = 0
stat("/tmp/logtest.txt.1", {st_mode=S_IFREG|0644, st_size=943718535, ...}) = 0
unlink("/tmp/logtest.txt.1")            = 0
stat("/tmp/logtest.txt", {st_mode=S_IFREG|0644, st_size=943718535, ...}) = 0
rename("/tmp/logtest.txt", "/tmp/logtest.txt.1") = 0
open("/tmp/logtest.txt", O_WRONLY|O_CREAT|O_APPEND|O_CLOEXEC, 0666) = 3
fcntl(3, F_SETFD, FD_CLOEXEC)           = 0
fstat(3, {st_mode=S_IFREG|0644, st_size=0, ...}) = 0
lseek(3, 0, SEEK_END)                   = 0
ioctl(3, TIOCGWINSZ, 0x7fffb9c0c190)    = -1 ENOTTY (Inappropriate ioctl for device)
lseek(3, 0, SEEK_CUR)                   = 0
ioctl(3, TIOCGWINSZ, 0x7fffb9c0c0b0)    = -1 ENOTTY (Inappropriate ioctl for device)
lseek(3, 0, SEEK_CUR)                   = 0
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fed00372000
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7feced771000
write(3, "2020-06-21 15:59:57,806 - __main"..., 314572844) = 314572844
munmap(0x7feced771000, 314576896)       = 0
write(3, "\n", 1)                       = 1
munmap(0x7fed00372000, 314576896)       = 0
select(0, NULL, NULL, NULL, {0, 100000}) = 0 (Timeout)
getpid()                                = 1
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fed00372000
mmap(NULL, 393220096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fece8c71000
mremap(0x7fece8c71000, 393220096, 314576896, MREMAP_MAYMOVE) = 0x7fece8c71000
munmap(0x7fed00372000, 314576896)       = 0
lseek(3, 0, SEEK_END)                   = 314572845
lseek(3, 0, SEEK_CUR)                   = 314572845
munmap(0x7fece8c71000, 314576896)       = 0
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fed00372000
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7feced771000
write(3, "2020-06-21 16:00:00,734 - __main"..., 314572844) = 314572844
````

观察后面的 stat() 调用，你可以看到，它正在获取 /tmp/logtest.txt.1 的状态。可以猜测，这是第一个日志回滚文件，而正在写的日志文件路径，则是 /tmp/logtest.txt。

```
[root@izbp1goz1ulmtus2vec80fz ~]# lsof -p 6648
python  6648 root    3w   REG  253,1 943718535   407853 /tmp/logtest.txt
```

这说明，这个进程打开了文件 /tmp/logtest.txt，并且它的文件描述符是 3 号，而 3 后面的 w ，表示以写的方式打开。以每次 300MB 的速度，在“疯狂”写日志，而日志文件的路径是 /tmp/logtest.txt。

