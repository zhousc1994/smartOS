# smartOS
## 一.入门
SmartOS支持操作系统虚拟化，虚拟出SmartMachine。和KVM类似，一个SmartMachine看上去就是一个完整的机器，
有自己的硬件（存储、网络和处理器）和操作系统以及库文件等。和KVM不同的是，没有格外的虚拟化层。这意味着不同的SmartMachine在同一硬件上共用操作系统（SmartOS）

## 1.为什么要推荐使用SmartMachine或SmartOS，而不是Linux呢？

**主要有以下几个原因：**

### 1）性能
    文件系统，SmartOS使用了ZFS，每个SmartMachine运行自己的ZFS数据集，每个Linux/Windows虚拟机有自己的ZFS卷。
    ZFS使用了Adjustable Replacement Cache (ARC)算法，在缓存文件系统数据或元数据方面做得非常好
### 2） 可观测性（Observability）
    SmartOS提供了一个工具叫做DTrace
    DTrace机制可以用于：
    
    调试排错 —— 比如，使用DTrace跟踪函数的入口和返回，输出相应的参数和返回值。还可以用它研究某个函数是如何被调用的（stacktrace），对于某个给定事件，代码运行路径是怎样的等等。
    性能分析 —— DTrace允许你拿到纳秒级的信息。你可以用它去采样来看CPU是如何耗时的，也可以用来研究为什么应用程序会被阻止继续执行，会持续多久等。
    代码覆盖 —— DTrace可以判断程序是否有被执行，这在测试中很重要。

### 3） 可靠性
    SmartOS间接地来自于Solaris，而后者一直以来在可靠和安全方面都广受好评。此外，ZFS校验和可保证数据不受损坏，使用写时复制（copy-on-write）机制确保活跃数据不会被被覆盖。
    ZFS文件系统有良好的一致性，自从2006年就被用于生产环境，易于管理。最近还被用于大数据方面
    
## 二.smartOS概述

[1.虚拟化](./src/虚拟化.md)

[2.Zone的概念](./src/zones.md)

[3.smartOS安装](./src/zones.md)

## 三.常用操作

[1.如何创建HVM区](./src/operation/KVM.md)
