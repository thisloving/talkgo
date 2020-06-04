## iowait 分析

```
# 先删除上次启动的案例 
docker rm -f app
# 重新运行案例
docker run --privileged --name=app -itd feisky/app:iowait
```

dstat ，可以同时查看 CPU 和 I/O 这两种资源的使用情况

```
[root@izbp1goz1ulmtus2vec80fz /]# dstat 1 10
You did not select any stats, using -cdngy by default.
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
  0   0  99   0   0   0| 562B 2751B|   0     0 |   0     0 |  52   871 
  0   1  99   0   0   0|   0     0 |  60B  106B|   0     0 | 562   949 
  1   0  99   0   0   0|   0     0 |  60B  874B|   0     0 | 532   942 
  0   0 100   0   0   0|   0   328k| 120B  692B|   0     0 | 572   974 
  1   1  98   0   0   0|   0     0 |  60B 1732B|   0     0 | 518   925 
  0   0 100   0   0   0|   0     0 | 120B  692B|   0     0 | 545   953 
  1   1  98   0   0   0|   0     0 |  60B  624B|   0     0 | 537   944 
  0   0 100   0   0   0|   0     0 |  60B  346B|   0     0 | 567   975 
  1   1  98   0   0   0|   0     0 |  60B  346B|   0     0 | 533   967 
  1   0  99   0   0   0|   0     0 |  60B  692B|   0     0 | 553   958 
  0   1  99   0   0   0|   0     0 |  60B   74B|   0     0 | 534   944 
```

看上图，iowait 的升高跟磁盘的读写请求有关，很可能就是磁盘读导致的

 这部分实验未成功，有时间切换到ubuntu下试一试