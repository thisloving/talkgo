## 案例准备

```
[root@izbp1goz1ulmtus2vec80fz ~]# docker run --name=app -p 10000:80 -itd feisky/word-pop 
Unable to find image 'feisky/word-pop:latest' locally
latest: Pulling from feisky/word-pop
4fe2ade4980c: Already exists 
7cf6a1d62200: Already exists 
d16bb326b4ba: Already exists 
ed7df901e9a8: Already exists 
d7e55b1728aa: Already exists 
b1af5c7d1a24: Pull complete 
1c9c27752eff: Pull complete 
Digest: sha256:7e7ce343e6b6a7b4882cb67b9cd66283f1f11c3e840766fe30be8d31bceca587
Status: Downloaded newer image for feisky/word-pop:latest
cdaea69e981d31c6b00efe19cb0d0f013074982477ca23dced5889e47e512932
[root@izbp1goz1ulmtus2vec80fz ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
cdaea69e981d        feisky/word-pop     "python /app.py"         9 seconds ago       Up 8 seconds        0.0.0.0:10000->80/tcp     app
27c9a143ca44        nginx               "/docker-entrypoint.…"   2 weeks ago         Up 2 weeks          0.0.0.0:80->80/tcp        nginx
d1e8af279217        feisky/php-fpm:sp   "php-fpm -F --pid /o…"   3 weeks ago         Up 3 weeks                                    phpfpm
e69524aca360        centos:latest       "/bin/bash"              7 months ago        Up 7 months         0.0.0.0:6390->6390/tcp    myredis2nd
0d2fdd62f258        centos:latest       "/bin/bash"              7 months ago        Up 7 months         0.0.0.0:33306->3306/tcp   mycentos4th
4d138db3792d        centos:latest       "/bin/bash"              7 months ago        Up 7 months                                   mycentos2nd
```



```
[root@izbp1goz1ulmtus2vec80fz ~]# curl http://192.168.0.10:1000/popularity/word 
```

稍等一会儿，你会发现，这个接口居然这么长时间都没响应，究竟是怎么回事呢？我们先回到终端一来分析一下。

```
[root@izbp1goz1ulmtus2vec80fz ~]#  while true; do time curl http://192.168.0.10:10000/popularity/word; sleep 1; done  
```

实验下来，top观察这部分iowait并不高，没有发现作者说的问题

```
Tasks:  94 total,   1 running,  93 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883492 total,   516896 free,   264172 used,  1102424 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used.  1419360 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                              
 7842 root      20   0   96780  19804   3928 S  0.3  1.1   0:01.24 python                                                                                               
24144 root      10 -10  158932  38176   1644 S  0.3  2.0 568:51.42 AliYunDun                                                                                            
    1 root      20   0   51732   2800   1420 S  0.0  0.1  62:42.00 systemd  
```

这个章节实验不算成功。