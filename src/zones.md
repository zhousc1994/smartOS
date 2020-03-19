# 一.Zones
## 1.定义与特性（类似于容器的概念）
    1.Zones是SmartOS的虚拟实例，不同的Zones之间，相互隔离
    2.每一个区域共享一个资源池和单个操作系统内核
    3.与虚拟机不同的是，Zones是共享基本的OS内核！而每个虚拟机都有自己的OS内核
    


## 2. Global Zone(全局Zones)

    在SmartOS中，Global Zone是所有其他Zone的父级，并且充当SmartOS的管理层
    可以使用Global Zone来监视和管理非Global Zone的系统运行状况，容量和吞吐量

**Golobal Zone实际上是只读的虚拟机管理程序
  因为SmartOS不在本地安装。它可以直接从USB，CD-ROM或PXE引导到大多数只读环境中。
  引导至SmartOS时，实际上是将其加载到内存中并从ramdisk上运行。
  这意味着您通常要在Golobal Zone中进行的大多数更改将不会保留或根本不允许。例如：**

+ 无法添加用户。
+ 无法修改/usr下的文件。
+ 不能在/root,/etc下永久存储文件
+ 对SMF服务所做的任何更改都会在每次重新引导时重置。
    
**可以通过三个基本步骤来控制和实例化Zone**

+ Configure:定义区域的网络和权限集。
+ Install packages:构造，安装和初始化Zones的软件包
+ Run:引导区域并启动服务

## 3.Golobal Zone 与Zone的区别
    1.Golobal Zone是在启动并登录SmartOS时启动的Zone。您可以将Golobal Zone视为“基本操作系统”。
    2.Zones是您通过Golobal Zone管理的各种OS和KVM实例
    
## 4.Process Security(安全处理)
**Zone之间数据与命名空间分离**

+ Zone无法向其他Zone出信息
+ zone提供服务隔离和封装，保证安全性

## 5.Resource Security（资源安全）
+ 控制文件系统：从Golobal Zone挂载Zone，并且Golobal Zone管理员可以将Zone文件系统挂载为读/写或只读。除非Golobal Zone管理员允许，否则Zone中的进程无法重新挂载文件系统。

+ 控制网络堆栈：Golobal Zone管理员可以通过以下两种方式之一控制区域的IP行为：通过共享IP堆栈或专用IP堆栈。默认情况下，在SmartOS上，每个Zone都有其自己的IP堆栈。该Zone中的管理员将具有一组有限的特权，可以执行某些管理功能，例如选择IP地址和配置路由。

+ 命名空间分隔：Zone提供系统和网络配置名称空间的分隔，从而使每个区域都具有唯一的命名空间。内核管理的所有资源都通过多实例化维护，这意味着相同的资源将作为每个Zone中的不同实例进行管理，从而消除了命名空间冲突。例如，每个区域都维护的唯一实例/tmp。

+ 虚拟硬件：  在SmartOS区域上，无权访问任何物理设备。区域中的所有硬件组件都已完全虚拟化，包括网络连接
   



    
    