## Swap 原理

Swap 说白了就是把一块磁盘空间或者一个本地文件（以下讲解以磁盘为例），当成内存来使用。它包括换出和换入两个过程。

+ 所谓换出，就是把进程暂时不用的内存数据存储到磁盘中，并释放这些数据占用的内存。
+ 而换入，则是在进程再次访问这些内存的时候，把它们从磁盘读到内存中来。

Swap 其实是把系统的可用内存变大了。

kswapd0 定义了三个内存阈值（watermark，也称为水位），分别是页最小阈值（pages_min）、页低阈值（pages_low）和页高阈值（pages_high）。剩余内存，则使用 pages_free 表示。

+ 剩余内存小于页最小阈值，说明进程可用内存都耗尽了，只有内核才可以分配内存。
+ 剩余内存落在页最小阈值和页低阈值中间，说明内存压力比较大，剩余内存不多了。这时 kswapd0 会执行内存回收，直到剩余内存大于高阈值为止。
+ 剩余内存落在页低阈值和页高阈值中间，说明内存有一定压力，但还可以满足新内存请求。
+ 剩余内存大于页高阈值，说明剩余内存比较多，没有内存压力。

页最小阈值、页低阈值和页高阈值，都可以通过内存域在 proc 文件系统中的接口 /proc/zoneinfo 来查看

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/zoneinfo 
Node 0, zone      DMA
  pages free     1940
        min      95
        low      118
        high     142
        scanned  0
        spanned  4095
        present  3998
        managed  3977
    nr_free_pages 1940
    nr_alloc_batch 24
    nr_inactive_anon 24
    nr_active_anon 677
    nr_inactive_file 1109
    nr_active_file 148
    nr_unevictable 0
    nr_mlock     0
    nr_anon_pages 165
    nr_mapped    159
    nr_file_pages 1281
    nr_dirty     0
    nr_writeback 0
    nr_slab_reclaimable 18
    nr_slab_unreclaimable 20
    nr_page_table_pages 9
    nr_kernel_stack 5
    nr_unstable  0
    nr_bounce    0
    nr_vmscan_write 1
    nr_vmscan_immediate_reclaim 3
    nr_writeback_temp 0
    nr_isolated_anon 0
    nr_isolated_file 0
    nr_shmem     24
    nr_dirtied   180579
    nr_written   205872
    numa_hit     1822777
    numa_miss    0
    numa_foreign 0
    numa_interleave 0
    numa_local   1822777
    numa_other   0
    workingset_refault 13203
    workingset_activate 2824
    workingset_nodereclaim 9
    nr_anon_transparent_hugepages 1
    nr_free_cma  0
        protection: (0, 1823, 1823, 1823)
  pagesets
    cpu: 0
              count: 0
              high:  0
              batch: 1
  vm stats threshold: 2
  all_unreclaimable: 0
  start_pfn:         1
  inactive_ratio:    1
Node 0, zone    DMA32
  pages free     57231
        min      11168
        low      13960
        high     16752
        scanned  0
        spanned  520160
        present  520160
        managed  466896
    nr_free_pages 57231
    nr_alloc_batch 289
    nr_inactive_anon 4522
    nr_active_anon 48657
    nr_inactive_file 235207
    nr_active_file 87747
    nr_unevictable 0
    nr_mlock     0
    nr_anon_pages 43966
    nr_mapped    17304
    nr_file_pages 327564
    nr_dirty     0
    nr_writeback 0
    nr_slab_reclaimable 18503
    nr_slab_unreclaimable 5251
    nr_page_table_pages 1547
    nr_kernel_stack 213
    nr_unstable  0
    nr_bounce    0
    nr_vmscan_write 181
    nr_vmscan_immediate_reclaim 1908
    nr_writeback_temp 0
    nr_isolated_anon 0
    nr_isolated_file 0
    nr_shmem     4610
    nr_dirtied   62792932
    nr_written   73908447
    numa_hit     517597707
    numa_miss    0
    numa_foreign 0
    numa_interleave 12041
    numa_local   517597707
    numa_other   0
    workingset_refault 5524647
    workingset_activate 1233304
    workingset_nodereclaim 17263
    nr_anon_transparent_hugepages 9
    nr_free_cma  0
        protection: (0, 0, 0, 0)
  pagesets
    cpu: 0
              count: 111
              high:  186
              batch: 31
  vm stats threshold: 10
  all_unreclaimable: 0
  start_pfn:         4096
  inactive_ratio:    3
```

+ pages 处的 min、low、high，就是上面提到的三个内存阈值，而 free 是剩余内存页数，它跟后面的 nr_free_pages 相同。
+ nr_zone_active_anon 和 nr_zone_inactive_anon，分别是活跃和非活跃的匿名页数。
+ nr_zone_active_file 和 nr_zone_inactive_file，分别是活跃和非活跃的文件页数。

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/sys/vm/zone_reclaim_mode
0
```

默认的 0 ，表示既可以从其他 Node 寻找空闲内存，也可以从本地回收内存。

1、2、4 都表示只回收本地内存，2 表示可以回写脏数据回收内存，4 表示可以用 Swap 方式回收内存。

## swappiness

+ 对文件页的回收，当然就是直接回收缓存，或者把脏页写回磁盘后再回收。
+ 而对匿名页的回收，其实就是通过 Swap 机制，把它们写入磁盘后再释放内存。

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/sys/vm/swappiness 
0
```

swappiness 的范围是 0-100，数值越大，越积极使用 Swap，也就是更倾向于回收匿名页；数值越小，越消极使用 Swap，也就是更倾向于回收文件页。

## 小结

在内存资源紧张时，Linux 通过直接内存回收和定期扫描的方式，来释放文件页和匿名页，以便把内存分配给更需要的进程使用。

+ 文件页的回收比较容易理解，直接清空，或者把脏数据写回磁盘后再释放。
+ 而对匿名页的回收，需要通过 Swap 换出到磁盘中，下次访问时，再从磁盘换入到内存中。

可以设置 /proc/sys/vm/min_free_kbytes，来调整系统定期回收内存的阈值（也就是页低阈值），还可以设置 /proc/sys/vm/swappiness，来调整文件页和匿名页的回收倾向。