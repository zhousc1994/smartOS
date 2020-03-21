# 如何创建HVM区
    SmartOS具有两个不同的虚拟机监视器：KVM和 Bhyve，统称为HVM
    
## 1.编写json文件，主要描述KVM虚机的配置！
    创建json文件
    cd /opt/
    vim testvkm.json
  
    {
      "brand": "kvm",   // 虚机类型   kvm,bhyve
      "vcpus": 1,       // cpu核数         
      "autoboot": false,      // 自动启动
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
          "nic_tag": "admin",  // 名称
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
## 6.vmadm常用操作
### 1.vmadm 查询操作
**使用-o改变输出字段或-s更改排序键(-s -ram表示降序,不加-代表升序)**
`[root@headnode (bh1-kvm1:0) ~]# vmadm list -o uuid,type,ram,quota,cpu_shares,zfs_io_priority -s -ram`

**过滤虚拟机(type=OS(类型为OS的) ram=~^1(过滤规则))**
`[root@headnode (bh1-kvm1:0) ~]# vmadm list -o uuid,type,ram,quota,cpu_shares,zfs_io_priority -s -ram,cpu_shares type=OS ram=~^1`

**使用-p分割字段（:），nics.0.ip表示获取第一个NIC的ip地址**

    [root@headnode (bh1-kvm1:0) ~]# vmadm list -p -o uuid,nics.0.ip
    ad170576-3ad3-e201-cd89-ad4260c3a3ce:10.0.0.1
### 2.获取实例属性
    vmadm get 54f1cc77-68f1-42ab-acac-5c4f64
**使用lookup命令(查询别名为\*zone7\*的虚机)**

    [root@headnode (bh1-kvm1:0) ~]# vmadm lookup alias=~zone7
    54f1cc77-68f1-42ab-acac-5c4f64f5d6e0
    
    // 通过-j选项不需要知道ID直接查看json文件
    [root@headnode (bh1-kvm1:0) ~]# vmadm lookup -j alias=~zone7
    
    // 合并过滤器  以'a'或'b'开头的别名查找所有128M实例
    [root@headnode (bh1-kvm1:0) ~]# vmadm lookup ram=128 alias=~^[ab]
    5c12a3ef-e60c-479a-a6be-ba93712a3893
    29bdabe1-ff9e-4387-9316-39961d5dbe5c
    20de2bfc-56de-4cd9-8e25-80b017615788

### 3.修改实例
    // 直接修改
    [root@headnode (bh1-kvm1:0) ~]# vmadm update 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0 quota=40
    
    //使用JSON输入进行更新
    [root@headnode (bh1-kvm1:0) ~]# echo '{"cpu_shares": 100}' | vmadm update 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0

**其他修改**

    某些字段（例如nics和磁盘）无法直接更新。数组字段通常是这种情况。他们需要指定一个操作。

#### 修改网卡add_nics，remove_nics
**添加网卡**

    [root@headnode (bh1-kvm1:0) ~]# cat add_nic.json
    {
        "add_nics": [
            {
                "physical": "net1",
                "index": 1,
                "nic_tag": "external",
                "mac": "b2:1e:ba:a5:6e:71",
                "ip": "10.2.121.71",
                "netmask": "255.255.0.0",
                "gateway": "10.2.121.1"
            }
        ]
    }
    [root@headnode (bh1-kvm1:0) ~]# cat add_nic.json | vmadm update 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0
    Successfully updated 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0
**本示例用于update_nics修改nic的IP。NIC由标识mac**

    [root@headnode (bh1-kvm1:0) ~]# cat update_nic.json
    {
        "update_nics": [
            {
                "mac": "b2:1e:ba:a5:6e:71",
                "ip": "10.2.121.72"
            }
        ]
    }
    [root@headnode (bh1-kvm1:0) ~]# cat update_nic.json | vmadm update 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0
    Successfully updated 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0
    
**删除vm的网络**
    
    [root@headnode (bh1-kvm1:0) ~]# echo '{"remove_nics": ["b2:1e:ba:a5:6e:71"]}' | vmadm update 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0
    Successfully updated 54f1cc77-68f1-42ab-acac-5c4f64f5d6e0

