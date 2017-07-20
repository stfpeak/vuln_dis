## 什么是Tap设备

qemu在这里使用了Tap设备，那Tap设备是什么呢？

TUN/TAP虚拟网络设备的原理比较简单，在Linux内核中添加了一个TUN/TAP虚拟网络设备的驱动程序和一个与之相关连的字符设备/dev/net/tun，字符设备tun作为用户空间和内核空间交换数据的接口。当内核将数据包发送到虚拟网络设备时，数据包被保存在设备相关的一个队列中，直到用户空间程序通过打开的字符设备tun的描述符读取时，它才会被拷贝到用户空间的缓冲区中，其效果就相当于，数据包直接发送到了用户空间。通过系统调用write发送数据包时其原理与此类似。

TUN/TAP驱动程序中包含两个部分，一部分是字符设备驱动，还有一部分是网卡驱动部分。利用网卡驱动部分接收来自TCP/IP协议栈的网络分包并发送或者反过来将接收到的网络分包传给协议栈处理，而字符驱动部分则将网络分包在内核与用户态之间传送，模拟物理链路的数据接收和发送。Tun/tap驱动很好的实现了两种驱动的结合。

总而言之，Tap设备实现了这么一种能力，对于用户空间来讲实现了网络数据包在内核态与用户态之间的传输，对于内核管理来说，它是一个网络设备或者直接呈现的是一个网络接口，可以像普通网络接口一样进行收发包。

Tap设备的创建

当命令行中通过-net tap指明创建Tap设备后，在Qemu的main函数中首先解析到了tap的参数选项，然后进入了设备创建流程：#
```c
main()  file: vl.c, line: 2345
    net_init_clients()  file: net.c, line: 991
        net_init_client()  file: net.c, line: 962
            net_client_init()  file: net.c, line: 701
                net_client_init1()  file: net.c, line: 628
                    net_client_init_fun[opts->kind](opts, name, peer)
```

net_tap_init()主要做了两件事：

1. 通过tap_open(){open(“/dev/net/tun”)}返回了Tap设备的文件描述符fd;
2. 将Tap设备的文件描述符加入Qemu事件监听列表;

## 参考
[Qemu之Network Device全虚拟方案一:前端网络流的建立](http://royluo.org/2014/07/17/netdev-virtual-1/)
