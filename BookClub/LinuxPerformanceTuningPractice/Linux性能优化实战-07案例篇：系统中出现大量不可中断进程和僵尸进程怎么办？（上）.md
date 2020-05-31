## 进程状态

+ R 是 Running 或 Runnable 的缩写，表示进程在 CPU 的就绪队列中，正在运行或者正在等待运行。
+ D 是 Disk Sleep 的缩写，也就是不可中断状态睡眠（Uninterruptible Sleep），一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程或中断打断。
+ Z 是 Zombie 的缩写，如果你玩过“植物大战僵尸”这款游戏，应该知道它的意思。它表示僵尸进程，也就是进程实际上已经结束了，但是父进程还没有回收它的资源（比如进程的描述符、PID 等）。
+ S 是 Interruptible Sleep 的缩写，也就是可中断状态睡眠，表示进程因为等待某个事件而被系统挂起。当进程等待的事件发生时，它会被唤醒并进入 R 状态。
+ I 是 Idle 的缩写，也就是空闲状态，用在不可中断睡眠的内核线程上。前面说了，硬件交互导致的不可中断进程用 D 表示，但对某些内核线程来说，它们有可能实际上并没有任何负载，用 Idle 正是为了区分这种情况。要注意，D 状态的进程会导致平均负载升高， I 状态的进程却不会。
+ 暂停状态（Stopped），T**状态标志state的值为TASK_STOPPED。**当进程收到一个SIGSTOP信号后，就由运行状态进入停止状态，当受到一个SIGCONT信号时，又会恢复运行状态。这种状态主要用于程序的调试，**又被叫做“暂停状态”、“挂起状态”。**
+  X，也就是 Dead 的缩写，表示进程已经消亡，你不会在 top 或者 ps 命令中看到它。

## 僵尸进程

+ **僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。**
+ **孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。**

**任何一个子进程(init除外)在exit()之后，并非马上就消失掉，而是留下一个称为僵尸进程(Zombie)的数据结构，等待父进程处理。**这是每个 子进程在结束时都要经过的阶段。如果子进程在exit()之后，父进程没有来得及处理，这时用ps命令就能看到子进程的状态是“Z”。如果父进程能及时 处理，可能用ps命令就来不及看到子进程的僵尸状态，但这并不等于子进程不经过僵尸状态。  如果父进程在子进程结束之前退出，则子进程将由init接管。init将会以父进程的身份对僵尸状态的子进程进行处理。

　僵尸进程危害场景：

一旦父进程没有处理子进程的终止，还一直保持运行状态，那么子进程就会一直处于僵尸状态。大量的僵尸进程会用尽 PID 进程号，导致新进程不能创建

## 案例分析

// 示例代码（没有用稿件上的）

```
 #include <stdio.h>                                                               
 #include <unistd.h>                                                               
 #include <errno.h>                                                               
 #include <stdlib.h>                                                                                                                                             
int main()                                                                         
{                                                                                 
    int i = 0;                                                                     
    for (i = 0; i < 200; i++) {                                                   
        pid_t pid;                                                                 
        pid = fork();                                                             
        if (pid < 0)                                                               
        {                                                                         
            perror("fork error:");                                                 
            exit(1);                                                               
        }                                                                         
        else if (pid == 0)                                                         
        {                                                                         
           printf("I am child process.I am exiting.\n");                           
            exit(0);                                                               
        }                                                                          
        printf("I am father process.I will sleep two seconds\n");                 
        //等待子进程先退出                                                          
        sleep(1);                                                                 
        //输出进程信息                                                              
        system("ps -o pid,ppid,state,tty,command");                                                                                             
        printf("father process is exiting.\n");                                  
    }                                                                             
    return 0;                                                                     
}   
```

```
// top输出
[root@izbp1goz1ulmtus2vec80fz ~]# top
top - 21:55:03 up 784 days,  6:27, 10 users,  load average: 0.04, 0.03, 0.05
Tasks: 139 total,   1 running, 103 sleeping,   0 stopped,  35 zombie
%Cpu(s):  1.4 us,  1.7 sy,  0.0 ni, 96.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883492 total,    96120 free,  1082212 used,   705160 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   619032 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                              
 8896 root      20   0  108188   1128    740 S  0.7  0.1   0:52.38 pidstat                                                                                              
24144 root      10 -10  157904  41720   5212 S  0.7  2.2 335:24.59 AliYunDun                                                                                            
  255 root      20   0       0      0      0 S  0.3  0.0   1:40.62 jbd2/vda1-8                                                                                          
  564 root      20   0 1120740 825596   2652 S  0.3 43.8   1:12.91 perf                                                                                                 
    1 root      20   0   51732   2908   1528 S  0.0  0.2  58:16.91 systemd  
```

```
//ps查看结果
[root@izbp1goz1ulmtus2vec80fz ~]# ps x
  PID TTY      STAT   TIME COMMAND
  27268 pts/4    S+     0:00 ./zombie
27269 pts/4    Z+     0:00 [zombie] <defunct>
27272 pts/4    Z+     0:00 [zombie] <defunct>
27274 pts/4    Z+     0:00 [zombie] <defunct>
27277 pts/4    Z+     0:00 [zombie] <defunct>
27280 pts/4    Z+     0:00 [zombie] <defunct>
27284 pts/4    Z+     0:00 [zombie] <defunct>
27288 pts/4    Z+     0:00 [zombie] <defunct>
27292 pts/4    Z+     0:00 [zombie] <defunct>
27296 pts/4    Z+     0:00 [zombie] <defunct>
27301 pts/4    Z+     0:00 [zombie] <defunct>
27306 pts/4    Z+     0:00 [zombie] <defunct>
27311 pts/4    Z+     0:00 [zombie] <defunct>
27315 pts/4    Z+     0:00 [zombie] <defunct>
27320 pts/4    Z+     0:00 [zombie] <defunct>
27324 pts/4    Z+     0:00 [zombie] <defunct>
27329 pts/4    Z+     0:00 [zombie] <defunct>
27332 pts/4    Z+     0:00 [zombie] <defunct>
27334 pts/4    Z+     0:00 [zombie] <defunct>
27337 pts/4    Z+     0:00 [zombie] <defunct>
27340 pts/4    Z+     0:00 [zombie] <defunct>
27343 pts/4    Z+     0:00 [zombie] <defunct>
27346 pts/4    Z+     0:00 [zombie] <defunct>
27348 pts/4    Z+     0:00 [zombie] <defunct>
27351 pts/4    Z+     0:00 [zombie] <defunct>
27354 pts/4    Z+     0:00 [zombie] <defunct>
27357 pts/4    Z+     0:00 [zombie] <defunct>
27360 pts/4    Z+     0:00 [zombie] <defunct>
```

可以发现僵尸进程的状态是Z，并且占着进程号不释放。