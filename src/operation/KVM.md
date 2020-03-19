# 如何创建HVM区
    SmartOS具有两个不同的虚拟机监视器：KVM和 Bhyve，统称为HVM
    
## 1.编写json文件，主要描述KVM虚机的配置！
    创建json文件
    cd /opt/
    vim testvkm.json
  
    {
      "brand": "kvm",   // 虚机类型   kvm,bhyve
      "vcpus": 1,       // cpu核数         
      "autoboot": false,      // 自动boot
      "ram": 1024,             // 分配内存
      "resolvers": ["208.67.222.222", "208.67.220.220"],   // DNS
      "disks": [        // 硬盘，存储
        {
          "boot": true,
          "model": "virtio",
          "size": 40960     // 硬盘大小
        }
      ],
      "nics": [             //网络
        {
          "nic_tag": "admin",
          "model": "virtio",
          "ip": "10.88.88.51",
          "netmask": "255.255.255.0",
          "gateway": "10.88.88.1",
          "primary": 1
        }
      ]
    }
## 2.创建Hvm
 `vmadm create -f testvkm.json`
## 3. 查询所有的HVM列表
`vmadm list`

## 4.找到创建的HVM对应的文件目录，将IOS文件导入到对应目录
**查找vm对应文件系统的位置**

`zfs list`

**进入zone内部**

`cd /zones/b8ab5fc1-8576-45ef-bb51-9826b52a4651/root/`

**设置boot启动对应ios文件**

`vmadm boot b8ab5fc1-8576-45ef-bb51-9826b52a4651 order=cd,once=d cdrom=/debian.iso,ide`

## 5.使用VNC连接到创建的虚拟机
    $ vmadm info b8ab5fc1-8576-45ef-bb51-9826b52a4651 vnc
    {
      "vnc": {
        "display": 39565,
        "port": 45465,
        "host": "10.99.99.7"
      }
    }