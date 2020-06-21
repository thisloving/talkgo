##  索引节点和目录项

 为了方便管理，Linux 文件系统为每个文件都分配两个数据结构，索引节点（index node）和目录项（directory entry）。它们主要用来记录文件的元信息和目录结构。

+ 索引节点，简称为 inode，用来记录文件的元数据，比如 inode 编号、文件大小、访问权限、修改日期、数据的位置等。
+ 目录项，简称为 dentry，用来记录文件的名字、索引节点指针以及与其他目录项的关联关系。

通过硬链接为文件创建的别名，就会对应不同的目录项，不过这些目录项本质上还是链接同一个文件，所以，它们的索引节点相同。

第一，目录项本身就是一个内存缓存，而索引节点则是存储在磁盘中的数据。

第二，磁盘在执行文件系统格式化时，会被分成三个存储区域，超级块、索引节点区和数据块区。

## 虚拟文件系统

目录项、索引节点、逻辑块以及超级块，构成了 Linux 文件系统的四大基本要素。又引入了一个抽象层，也就是虚拟文件系统 VFS（Virtual File System）。VFS 定义了一组所有文件系统都支持的数据结构和标准接口。这样，用户进程和内核中的其他子系统，只需要跟 VFS 提供的统一接口进行交互就可以了，而不需要再关心底层各种文件系统的实现细节。

在 VFS 的下方，Linux 支持各种各样的文件系统，按照存储位置的不同，这些文件系统可以分为三类。

+ 第一类是基于磁盘的文件系统，也就是把数据直接存储在计算机本地挂载的磁盘中。常见的 Ext4、XFS、OverlayFS 等，都是这类文件系统。
+ 第二类是基于内存的文件系统，也就是我们常说的虚拟文件系统。这类文件系统，不需要任何磁盘分配存储空间，但会占用内存。我们经常用到的 /proc 文件系统，其实就是一种最常见的虚拟文件系统。
+ 第三类是网络文件系统，也就是用来访问其他计算机数据的文件系统，比如 NFS、SMB、iSCSI 等。

## 文件系统 I/O

第一种，根据是否利用标准库缓存，可以把文件 I/O 分为缓冲 I/O 与非缓冲 I/O。

+ 缓冲 I/O，是指利用标准库缓存来加速文件的访问，而标准库内部再通过系统调度访问文件。
+ 非缓冲 I/O，是指直接通过系统调用来访问文件，不再经过标准库缓存。

第二，根据是否利用操作系统的页缓存，可以把文件 I/O 分为直接 I/O 与非直接 I/O。

+ 直接 I/O，是指跳过操作系统的页缓存，直接跟文件系统交互来访问文件。想要实现直接 I/O，需要你在系统调用中，指定 O_DIRECT 标志。如果没有设置过，默认的是非直接 I/O。

+ 非直接 I/O 正好相反，文件读写时，先要经过系统的页缓存，然后再由内核或额外的系统调用，真正写入磁盘。

第三，根据应用程序是否阻塞自身运行，可以把文件 I/O 分为阻塞 I/O 和非阻塞 I/O：

+ 阻塞 I/O，是指应用程序执行 I/O 操作后，如果没有获得响应，就会阻塞当前线程，自然就不能执行其他任务。
+ 非阻塞 I/O，是指应用程序执行 I/O 操作后，不会阻塞当前的线程，可以继续执行其他的任务，随后再通过轮询或者事件通知的形式，获取调用的结果。

第四，根据是否等待响应结果，可以把文件 I/O 分为同步和异步 I/O：

+ 所谓同步 I/O，是指应用程序执行 I/O 操作后，要一直等到整个 I/O 完成后，才能获得 I/O 响应。

+ 所谓异步 I/O，是指应用程序执行 I/O 操作后，不用等待完成和完成后的响应，而是继续执行就可以。等到这次 I/O 完成后，响应会用事件通知的方式，告诉应用程序。

## 性能观测

```
[root@izbp1goz1ulmtus2vec80fz ~]# df /dev/vda1 
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/vda1       41151808 32554052   6484324  84% /
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# df -i /dev/vda1  
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/vda1      2621440 254423 2367017   10% /
```

当你发现索引节点空间不足，但磁盘空间充足时，很可能就是过多小文件导致的。所以，一般来说，删除这些小文件，或者把它们移动到索引节点充足的其他磁盘中，就可以解决这个问题。

## 缓存

free 输出的 Cache，是页缓存和可回收 Slab 缓存的和

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/meminfo | grep -E "SReclaimable|Cached" 
Cached:           584052 kB
SwapCached:            0 kB
SReclaimable:      63476 kB
```

```
[root@izbp1goz1ulmtus2vec80fz ~]# cat /proc/slabinfo | grep -E '^#|dentry|inode' 
# name            <active_objs> <num_objs> <objsize> <objperslab> <pagesperslab> : tunables <limit> <batchcount> <sharedfactor> : slabdata <active_slabs> <num_slabs> <sharedavail>
ext4_inode_cache   14985  14985   1032   15    4 : tunables    0    0    0 : slabdata    999    999      0
mqueue_inode_cache      9      9    896    9    2 : tunables    0    0    0 : slabdata      1      1      0
hugetlbfs_inode_cache     13     13    608   13    2 : tunables    0    0    0 : slabdata      1      1      0
inotify_inode_mark     46     46     88   46    1 : tunables    0    0    0 : slabdata      1      1      0
sock_inode_cache     300    348    640   12    2 : tunables    0    0    0 : slabdata     29     29      0
shmem_inode_cache   1039   1068    680   12    2 : tunables    0    0    0 : slabdata     89     89      0
proc_inode_cache    1644   1644    656   12    2 : tunables    0    0    0 : slabdata    137    137      0
inode_cache        15418  15418    592   13    2 : tunables    0    0    0 : slabdata   1186   1186      0
dentry             45381  45381    192   21    1 : tunables    0    0    0 : slabdata   2161   2161      0
selinux_inode_security  10098  10098     40  102    1 : tunables    0    0    0 : slabdata     99     99      0
```

dentry 行表示目录项缓存，inode_cache 行，表示 VFS 索引节点缓存，其余的则是各种文件系统的索引节点缓存。

```
[root@izbp1goz1ulmtus2vec80fz ~]# slabtop 
 Active / Total Objects (% used)    : 265926 / 277810 (95.7%)
 Active / Total Slabs (% used)      : 12130 / 12130 (100.0%)
 Active / Total Caches (% used)     : 75 / 101 (74.3%)
 Active / Total Size (% used)       : 80489.41K / 82458.27K (97.6%)
 Minimum / Average / Maximum Object : 0.01K / 0.30K / 8.00K

  OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME                   
 39186  39186 100%    0.57K   2799       14     22392K radix_tree_node
 14985  14985 100%    1.01K    999       15     15984K ext4_inode_cache
 15418  15418 100%    0.58K   1186       13      9488K inode_cache
 45444  45444 100%    0.19K   2164       21      8656K dentry
 49257  49257 100%    0.10K   1263       39      5052K buffer_head
  3920   3898  99%    0.50K    490        8      1960K kmalloc-512
   480    464  96%    4.00K     60        8      1920K kmalloc-4096
  9282   7531  81%    0.19K    442       21      1768K kmalloc-192
 15228  15228 100%    0.11K    423       36      1692K kernfs_node_cache
  1592   1470  92%    1.00K    199        8      1592K kmalloc-1024
  5360   4959  92%    0.25K    335       16      1340K kmalloc-256
 19328  16859  87%    0.06K    302       64      1208K kmalloc-64
   584    549  94%    2.00K     73        8      1168K kmalloc-2048
  1644   1644 100%    0.64K    137       12      1096K proc_inode_cache
   264    239  90%    3.95K     33        8      1056K task_struct
  4680   4219  90%    0.21K    260       18      1040K vm_area_struct
   276    252  91%    2.62K     23       12       736K task_xstate
  1068   1039  97%    0.66K     89       12       712K shmem_inode_cache
   270    270 100%    2.06K     18       15       576K idr_layer_cache
  9945   4299  43%    0.05K    117       85       468K shared_policy_node
 10098  10098 100%    0.04K     99      102       396K selinux_inode_security
   180    107  59%    2.06K     12       15       384K sighand_cache
   180    107  59%    2.12K     12       15       384K TCPv6
    40     40 100%    8.00K     10        4       320K kmalloc-8192
   348    300  86%    0.62K     29       12       232K sock_inode_cache
  1632   1508  92%    0.12K     51       32       204K kmalloc-128
   480    479  99%    0.38K     48       10       192K mnt_cache
  1974   1894  95%    0.09K     47       42       188K kmalloc-96
   154    132  85%    1.12K     11       14       176K signal_cache
  2091   2091 100%    0.08K     41       51       164K anon_vma
   100     64  64%    1.56K     10       10       160K mm_struct
  3876   3876 100%    0.04K     38      102       152K ext4_extent_status
```

