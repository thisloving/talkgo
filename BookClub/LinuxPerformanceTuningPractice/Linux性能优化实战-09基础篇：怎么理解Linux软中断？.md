### 除了 iowait，软中断（softirq）CPU 使用率升高也是最常见的一种性能问题。

```
中断是系统用来响应硬件设备请求的一种机制，它会打断进程的正常调度和执行，然后调用内核中的中断处理程序来响应设备的请求。
```

中断其实是一种异步的事件处理机制，可以提高系统的并发处理能力。

## 软中断

Linux 将中断处理过程分成了两个阶段，也就是上半部和下半部：

+ 上半部用来快速处理中断，它在中断禁止模式下运行，主要处理跟硬件紧密相关的或时间敏感的工作。
+ 下半部用来延迟处理上半部未完成的工作，通常以内核线程的方式运行。

书上举了网卡的例子，这个例子可以深入，了解网卡，tcp/ip协议、dma等。

并且每个 CPU 都对应一个软中断内核线程：

```
[root@izbp1goz1ulmtus2vec80fz /]#  ps aux | grep softirq
root         3  0.0  0.0      0     0 ?        S     2018   2:34 [ksoftirqd/0]
```

+ /proc/softirqs 提供了软中断的运行情况；

  ```
  [root@izbp1goz1ulmtus2vec80fz /]# cat /proc/softirqs 
                      CPU0       
            HI:         41
         TIMER: 2937845612
        NET_TX:         84
        NET_RX:   99791315
         BLOCK:          0
  BLOCK_IOPOLL:          0
       TASKLET:       1817
         SCHED:          0
       HRTIMER:          0
           RCU: 2437285912
  ```

  NET_RX 表示网络接收中断，而 NET_TX 表示网络发送中断。

  TASKLET 是最常用的软中断实现机制，每个 TASKLET 只运行一次就会结束 ，并且只在调用它的函数所在的 CPU 上运行。

+ /proc/interrupts 提供了硬中断的运行情况。

  ```
  [root@izbp1goz1ulmtus2vec80fz /]# cat /proc/interrupts 
             CPU0       
    0:        140   IO-APIC-edge      timer
    1:       1544   IO-APIC-edge      i8042
    4:         58   IO-APIC-edge      serial
    6:          3   IO-APIC-edge      floppy
    8:          0   IO-APIC-edge      rtc0
    9:          0   IO-APIC-fasteoi   acpi
   11:        167   IO-APIC-fasteoi   uhci_hcd:usb1, virtio3
   12:       2160   IO-APIC-edge      i8042
   14:          0   IO-APIC-edge      ata_piix
   15:          0   IO-APIC-edge      ata_piix
   24:          0   PCI-MSI-edge      virtio2-config
   25:    7515255   PCI-MSI-edge      virtio2-req.0
   26:          8   PCI-MSI-edge      virtio0-config
   27:   42321581   PCI-MSI-edge      virtio0-input.0
   28:          1   PCI-MSI-edge      virtio0-output.0
   29:          0   PCI-MSI-edge      virtio1-config
   30:        290   PCI-MSI-edge      virtio1-virtqueues
  NMI:          0   Non-maskable interrupts
  LOC: 2748979782   Local timer interrupts
  SPU:          0   Spurious interrupts
  PMI:          0   Performance monitoring interrupts
  IWI:  765108885   IRQ work interrupts
  RTR:          0   APIC ICR read retries
  RES:          0   Rescheduling interrupts
  CAL:          0   Function call interrupts
  TLB:          0   TLB shootdowns
  TRM:          0   Thermal event interrupts
  THR:          0   Threshold APIC interrupts
  DFR:          0   Deferred Error APIC interrupts
  MCE:          0   Machine check exceptions
  MCP:     227049   Machine check polls
  ERR:          0
  MIS:          0
  PIN:          0   Posted-interrupt notification event
  PIW:          0   Posted-interrupt wakeup event
  ```

