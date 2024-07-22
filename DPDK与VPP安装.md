---
title: DPDK与VPP安装
---


# 环境

OS：Ubuntu 22.04 https://developer.aliyun.com/mirror

VPP：22.10.1 https://github.com/FDio/vpp

DPDK：22.11 https://github.com/DPDK/dpdk

# 准备

换APT源：

```shell
root@ubuntu:/opt# vim /etc/apt/source.list
deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

更新APT源：

```shell
root@ubuntu:/opt# apt update
```

# VPP

必要软件安装：

```shell
root@ubuntu:/opt# apt install -y git make make-guile
```

下载源码：

```shell
root@ubuntu:/opt# git clone https://github.com/FDio/vpp.git
```

依赖安装（需要挂代理）：

```shell
root@ubuntu:/opt# cd vpp
root@ubuntu:/opt# make install-dep
root@ubuntu:/opt# make install-ext-deps

# 或者
root@ubuntu:/opt# cd vpp
root@ubuntu:/opt# ./extras/vagrant/build.sh
```

编译：

```shell
root@ubuntu:/opt# make build-release # build-release是正式版，build是debug版
root@ubuntu:/opt# make pkg-deb
```

安装：

```
root@ubuntu:/opt# dpkg -i build-root/*.deb
```

进入控制台：

```shell
root@ubuntu:/opt# vppctl
```

# DPDK

必要软件安装：

```shell
root@ubuntu:/opt# apt install -y meson python3-pyelftools pkg-config
```

下载源码：

```shell
root@ubuntu:/opt# git clone https://github.com/DPDK/dpdk.git
```

编译安装：

```shell
root@ubuntu:/opt# cd dpdk
root@ubuntu:/opt# meson  build
root@ubuntu:/opt# cd build
root@ubuntu:/opt# ninja
root@ubuntu:/opt# ninja install
```

编译 igb uio 驱动：

```shell
root@ubuntu:/opt# git clone http://dpdk.org/git/dpdk-kmods.git
root@ubuntu:/opt# cd dpdk-kmods/linux/igb_uio
root@ubuntu:/opt# make
```

验证环境是否OK：

```shell
root@ubuntu:/opt# cd examples/l2fwd
root@ubuntu:/opt# make
```

加载 igb_uio 驱动：

```shell
root@ubuntu:/opt# cd dpdk-kmods/linux/igb_uio
root@ubuntu:/opt# modprobe uio
root@ubuntu:/opt# insmod igb_uio.ko intr_mode=legacy
```

分配大页内存（这里是1G）：

```shell
echo 512 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

绑定网卡给DPDK：

```shell
root@ubuntu:/opt# dpdk-devbind.py -s # 查看所有网卡信息
root@ubuntu:/opt# ifconfig ens34 down
root@ubuntu:/opt# ifconfig ens35 down
root@ubuntu:/opt# dpdk-devbind.py -b igb_uio ens34 ens35
```

运行 l2fwd 程序：

```shell
root@ubuntu:/opt# cd examples/l2fwd/build/
root@ubuntu:/opt# ./l2fwd -l 0-1 -- -p 0x3 -T 1
```

如果看到以下信息，说明 DPDK 环境没问题！

```
EAL: Detected CPU lcores: 8
EAL: Detected NUMA nodes: 1
EAL: Detected shared linkage of DPDK
EAL: Multi-process socket /var/run/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'PA'
EAL: VFIO support initialized
EAL: Probe PCI driver: net_e1000_em (8086:100f) device: 0000:02:02.0 (socket 0)
EAL: Error reading from file descriptor 20: Input/output error
EAL: Probe PCI driver: net_e1000_em (8086:100f) device: 0000:02:03.0 (socket 0)
EAL: Error reading from file descriptor 6: Input/output error
TELEMETRY: No legacy callbacks, legacy socket not created
MAC updating enabled
Lcore 0: RX port 0 TX port 1
Lcore 1: RX port 1 TX port 0
Initializing port 0... EAL: Error enabling interrupts for fd 20 (Input/output error)
done: 
Port 0, MAC address: 00:0C:29:0C:53:91

Initializing port 1... EAL: Error enabling interrupts for fd 6 (Input/output error)
done: 
Port 1, MAC address: 00:0C:29:0C:53:9B


Checking link statusdone
Port 0 Link up at 1 Gbps FDX Autoneg
Port 1 Link up at 1 Gbps FDX Autoneg
L2FWD: entering main loop on lcore 1
L2FWD:  -- lcoreid=1 portid=1
L2FWD: entering main loop on lcore 0
L2FWD:  -- lcoreid=0 portid=0

Port statistics ====================================
Statistics for port 0 ------------------------------
Packets sent:                        0
Packets received:                    0
Packets dropped:                     0
Statistics for port 1 ------------------------------
Packets sent:                        0
Packets received:                    0
Packets dropped:                     0
Aggregate statistics ===============================
Total packets sent:                  0
Total packets received:              0
Total packets dropped:               0
====================================================
^C

Signal 2 received, preparing to exit...
EAL: Error disabling interrupts for fd 20 (Input/output error)
Closing port 0... Done
EAL: Error disabling interrupts for fd 6 (Input/output error)
Closing port 1... Done
Bye...
```

解绑网卡：

网卡 ens34 和 ens35 被 DPDK 占用后，ifconfig 里是没有的，要恢复请进行如下操作。

首先查看两个网卡 pci 设备号：

```shell
root@ubuntu2204:~# lspci | grep Eth
02:00.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)
02:01.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)
02:02.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)
02:03.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)

```

可看到最后两个网卡设备号是 02:02.0 和 02.03.0。

然后解绑两个网卡的 igb_uio 驱动，绑定 e1000 驱动：

```shell
root@ubuntu:/opt# dpdk-devbind.py -u 02:02.0 02:03.0
root@ubuntu:/opt# dpdk-devbind.py -b e1000  02:02.0 02:03.0
```

最后将网卡up起来：

```shell
root@ubuntu:/opt# ifconfig ens34 up
root@ubuntu:/opt# ifconfig ens35 up
```

**如果缺少动态链接库**

```shell
root@ubuntu:/opt# echo "/usr/local/lib" >> /etc/ld.so.conf
root@ubuntu:/opt# /sbin/ldconfig
```

# VPP+DPDK示例

- 环境准备

  本次demo两台机器，每台机器两张网卡，分别是NAT与HOST模式

  机器一为VPP：ens33：192.168.10.132/24（NAT）ens34（HOST）：192.168.2.15/24（后续会配置）

  机器二为PC1：ens33：192.168.10.129/24（NAT）ens34（HOST）：192.168.2.11/24（后续会配置）

- 配置PC1的IP

  Ubuntu 17.10及更高版本使用`Netplan`作为默认[网络管理](https://so.csdn.net/so/search?q=网络管理&spm=1001.2101.3001.7020)工具

  ```shell
  root@ubuntu:~# vim /etc/netplan/01-network-manager-all.yaml
  # Let NetworkManager manage all devices on this system
  network:
    version: 2
    renderer: NetworkManager
    ethernets:
      ens34:
        addresses: ["192.168.2.11/24"]
        #      gateway4: "192.168.2.1"
        dhcp4: false
        dhcp6: false
  ```

  编辑完执行以下命令生效

  ```shell
  root@ubuntu:~# netplan apply
  ```

- DPDK挂载网卡

  加载igb_uio驱动

  ```shell
  root@ubuntu:/opt# cd dpdk-kmods/linux/igb_uio
  root@ubuntu:/opt# modprobe uio
  root@ubuntu:/opt# insmod igb_uio.ko intr_mode=legacy
  ```

  将机器一的ens34网卡进行挂载

  ```shell
  root@ubuntu:/opt# dpdk-devbind.py -s # 查看挂载信息
  root@ubuntu:/opt# ifconfig ens34 down
  root@ubuntu:/opt# dpdk-devbind.py -b igb_uio ens34
  ```

- 验证是否挂载成功

  ```shell
  # 查看ens34是否从内核转移到DPDK
  root@ubuntu:/opt# dpdk-devbind.py -s
  # 查看vpp是否存在该网卡
  root@ubuntu:/opt# vppctl show interface
  ```

- 配置VPP

  ```shell
  # 编辑启动文件
  root@ubuntu:/opt# vim /etc/vpp//etc/vpp/startup.conf
  dpdk {
      dev 0000:02:02.0 {
          name eth34
      }
  }# 将dpdk挂载的网卡写入到VPP启动文件
  unix {
    nodaemon
    log /var/log/vpp/vpp.log
    full-coredump
    cli-listen /run/vpp/cli.sock
    gid vpp
  	startup-config /etc/vpp/setup-interface.txt # unix添加startup-config项目，以后启动VPP会自动加载该路径的配置
  }
  # 添加持久化配置
  root@ubuntu:/opt# vim /etc/vpp/setup-interface.txt
  set interface state eth34 up
  set interface ip address eth34 192.168.2.15/24
  ip route add 192.168.2.0/24 via 192.168.2.15 eth34
  ```

- 重启VPP

  ```shell
  root@ubuntu:/opt# systemctl restart vpp
  ```

- 连通性校验

  PC1 ping VPP

  ```shell
  root@ubuntu:~# ping 192.168.2.15
  PING 192.168.2.15 (192.168.2.15) 56(84) bytes of data.
  64 bytes from 192.168.2.15: icmp_seq=1 ttl=64 time=0.313 ms
  64 bytes from 192.168.2.15: icmp_seq=2 ttl=64 time=0.252 ms
  64 bytes from 192.168.2.15: icmp_seq=3 ttl=64 time=0.323 ms
  ```

  VPP ping PC1

  ```shell
  root@ubuntu:~# vppctl ping 192.168.2.11
  116 bytes from 192.168.2.11: icmp_seq=1 ttl=64 time=.6665 ms
  116 bytes from 192.168.2.11: icmp_seq=2 ttl=64 time=.2812 ms
  116 bytes from 192.168.2.11: icmp_seq=3 ttl=64 time=.2628 ms
  ```

  

# SSLProxy

```
libevent-dev
libsqlite3-dev
```

```shell
# 生成证书私钥
openssl genrsa -out ca.key 2048
# 生成证书
openssl req -new -x509 -days 1096 -key ca.key -out ca.crt
```

# clash

```shell
root@ubuntu:/opt wget https://github.com/Dreamacro/clash/releases/download/v1.13.0/clash-linux-amd64-v1.13.0.gz
root@ubuntu:/opt gunzip clash-linux-amd64-v1.13.0.gz
root@ubuntu:/opt mv clash-linux-amd64-v1.13.0.gz clash
root@ubuntu:/opt wget -O config.yaml [订阅链接]
root@ubuntu:/opt chmod +x clash
root@ubuntu:/opt ./chash -d .
```

