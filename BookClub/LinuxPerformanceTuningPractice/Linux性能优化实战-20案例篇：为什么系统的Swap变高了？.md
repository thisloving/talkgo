##  案例

Linux 本身支持两种类型的 Swap，即 Swap 分区和 Swap 文件。

```
// 创建Swap文件
[root@izbp1goz1ulmtus2vec80fz ~]# fallocate -l 8G /mnt/swapfile 
//修改权限只有根用户可以访问
[root@izbp1goz1ulmtus2vec80fz ~]# chmod 600 /mnt/swapfile
//配置Swap文件
[root@izbp1goz1ulmtus2vec80fz ~]# mkswap /mnt/swapfile
Setting up swapspace version 1, size = 8388604 KiB
no label, UUID=6978dcfd-4b54-47d5-a02d-01a9647e9651
//开启Swap
[root@izbp1goz1ulmtus2vec80fz ~]# swapon /mnt/swapfile
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1883492      221452      246408       18536     1415632     1454152
Swap:       8388604           0     8388604
```

free  输出中，Swap 空间以及剩余空间都从 0 变成了 8GB，说明 Swap 已经正常开启。

```
[root@izbp1goz1ulmtus2vec80fz ~]# dd if=/dev/vda1 of=/dev/null bs=1G count=2048  
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# sar -r -S 1
07:12:24 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
07:12:24 PM     66676   1816816     96.46    434292     39776   2916632     28.39    928964    769824       136

07:12:24 PM kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
07:12:24 PM   8388604         0      0.00         0      0.00
```

+ kbcommit，表示当前系统负载需要的内存。它实际上是为了保证系统内存不溢出，对需要内存的估计值。%commit，就是这个值相对总内存的百分比。
+ kbactive，表示活跃内存，也就是最近使用过的内存，一般不会被系统回收。
+ kbinact，表示非活跃内存，也就是不常访问的内存，有可能会被系统回收。

刚开始，剩余内存（kbmemfree）不断减少，而缓冲区（kbbuffers）则不断增大，由此可知，剩余内存不断分配给了缓冲区。一段时间后，剩余内存已经很小，而缓冲区占用了大部分内存。这时候，Swap 的使用开始逐渐增大，缓冲区和剩余内存则只在小范围内波动。

```
[root@izbp1goz1ulmtus2vec80fz ~]# for file in /proc/*/status ; do awk '/VmSwap|Name|^Pid/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 3 -n -r | head
writeback 15 
watchdog/0 10 
tuned 30181 0 kB
ttm_swap 238 
systemd-udevd 19213 0 kB
systemd-logind 474 0 kB
systemd-journal 324 0 kB
systemd 1 0 kB
su 6881 0 kB
su 1376 0 kB
```

## 小结

在内存资源紧张时，Linux 会通过 Swap ，把不常访问的匿名页换出到磁盘中，下次访问的时候再从磁盘换入到内存中来。你可以设置 /proc/sys/vm/min_free_kbytes，来调整系统定期回收内存的阈值；也可以设置 /proc/sys/vm/swappiness，来调整文件页和匿名页的回收倾向。