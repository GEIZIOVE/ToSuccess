#  Docker基础 - Docker网络使用和配置

> 上文已经向你介绍了，web容器创建和容器互联了，但是容器之间为什么可以直接通信？主机和容器之间为何可以通信？如何进行自定义的配置呢？所以这节就是我们要讲述的Docker网络。@pdai

## 单机中docker网络

### [¶](#理解docker-默认网桥) 理解Docker 默认网桥

> 为何上文的例子中，容器之间通过--link就可以相互通信，主机和容器之间也能通信？

在你安装Docker 服务默认会创建一个 `docker0` 网桥（其上有一个 docker0 内部接口），它在内核层连通了其他的物理或虚拟网卡，**这就将所有容器和本地主机都放到同一个物理网络。**

我们可用 docker network ls 命令查看：

```bash
[root@pdai ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b8c5abdb0bec        bridge              bridge              local
84e86bc93121        host                host                local
8e521527a897        none                null                local
```



Docker 安装时会自动在 host 上创建三个网络：`none，host，和bridge`。我们看下docker0 网桥：

> (brctl可以通过`yum install bridge-utils`安装)

```bash
[root@pdai ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242703f9d02       no              veth0004826
                                                        veth4ad3278
[root@pdai ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:08:c1:ea brd ff:ff:ff:ff:ff:ff
    inet 172.31.165.194/20 brd 172.31.175.255 scope global dynamic eth0
       valid_lft 310072401sec preferred_lft 310072401sec
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:70:3f:9d:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
45: veth4ad3278@if44: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether c2:77:e4:ea:f1:33 brd ff:ff:ff:ff:ff:ff link-netnsid 0
49: veth0004826@if48: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 6e:c9:1a:7c:18:b1 brd ff:ff:ff:ff:ff:ff link-netnsid 1

[root@pdai ~]# ip route
default via 172.31.175.253 dev eth0
169.254.0.0/16 dev eth0 scope link metric 1002
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
172.31.160.0/20 dev eth0 proto kernel scope link src 172.31.165.194
   
```



再用`docker network inspect`指令查看bridge网络：其Gateway就是网卡/接口docker0的IP地址：172.17.0.1。

```bash
[root@pdai ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b8c5abdb0becacfa1bfa1d72e2e663fb0157b62a9b8bee37e2607211722713cc",
        "Created": "2020-02-17T14:10:10.424119543+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1cbc9aeba2a8a826d460ecb49de17ddf8ac336e150c752a3c762fd38a3e15254": {
                "Name": "web",
                "EndpointID": "adb47ed0c60c6b80a442c71a5f35d63378cecca9598e0cef8409a6719552f4c2",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "d992e3c761e00649eb436b88c737adc54093b76119af0fb7878596b523f743ca": {
                "Name": "db",
                "EndpointID": "6a3dab5c545dd26e0ca6e36d928a32fd0a6197c8dbf4eeb718a4560e219575ed",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

```

从上面你可以看到bridge的配置信息和容器信息。

### [¶](#理解容器创建时的ip分配) 理解容器创建时的IP分配

> 为了理解容器创建时的IP分配，这里需要清理所有已经启动的环境，然后再启动容器，看前后对比

- 我们**清理所有容器**实例，下面展示的就是docker安装之后的, 注意和上面的对比下：

```bash
[root@pdai ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@pdai ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b8c5abdb0bec        bridge              bridge              local
84e86bc93121        host                host                local
8e521527a897        none                null                local
[root@pdai ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b8c5abdb0becacfa1bfa1d72e2e663fb0157b62a9b8bee37e2607211722713cc",
        "Created": "2020-02-17T14:10:10.424119543+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@pdai ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:08:c1:ea brd ff:ff:ff:ff:ff:ff
    inet 172.31.165.194/20 brd 172.31.175.255 scope global dynamic eth0
       valid_lft 310069965sec preferred_lft 310069965sec
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:70:3f:9d:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[root@pdai ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242703f9d02       no  
```

 
 

- **创建容器**

Docker 在创建一个容器的时候，会执行如下操作：

- 创建一对虚拟接口/网卡，也就是veth pair，分别放到本地主机和新容器中；
- 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 vethxxxxx；
- 容器一端放到新容器中，并修改名字作为 eth0，这个网卡/接口只在容器的名字空间可见；
- 从网桥可用地址段中（也就是与该bridge对应的network）获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 vethxxxx。

让我们启动一个容器，看下变化：

```bash
[root@pdai ~]# docker run -d --name db training/postgres
0ffd1092cd962bdbe335ce042b93d0f2082559600cacc82bbef40b8b66395e57
[root@pdai ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242703f9d02       no              vethd93e2ad
[root@pdai ~]# ip a | grep vethd93e2ad
51: vethd93e2ad@if50: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
[root@pdai ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b8c5abdb0becacfa1bfa1d72e2e663fb0157b62a9b8bee37e2607211722713cc",
        "Created": "2020-02-17T14:10:10.424119543+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0ffd1092cd962bdbe335ce042b93d0f2082559600cacc82bbef40b8b66395e57": {
                "Name": "db",
                "EndpointID": "a90cb50031effc99b9254fe4f1231bfbac8c4bb23d94c5a1425c1e116ac452dc",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
  
        @pdai: 代码已经复制到剪贴板
    
 
```

如果不指定--network，创建的容器默认都会挂到 docker0 上，使用本地主机上 docker0 接口的 IP 作为所有容器的默认网关

当有多个容器创建后，容器网络拓扑结构如下：

![img](E:\Development\Typora\images\docker-y-4.jpg)

### [¶](#理解容器和docker0的虚拟网卡的配对) 理解容器和docker0的虚拟网卡的配对

> 在上图中容器中eth0是怎么和host中虚拟网卡配对上的呢？

```bash
[root@pdai ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
0ffd1092cd96        training/postgres   "su postgres -c '/us…"   11 minutes ago      Up 11 minutes       5432/tcp            db
[root@pdai ~]# docker exec -it 0ffd1092cd96 /bin/bash
root@0ffd1092cd96:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
50: eth0@if51: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
  
```



我们可以看到host上`51: vethd93e2ad@if50`对应着容器中`50: eth0@if51`; 即host中index=51的接口/网卡vethd93e2ad的peer inferface index是50, container中index=50的网卡eth0的peer interface index是51。

可以利用ethtool来确认这种对应关系：分别在host和container中运行指令`ethtool -S <interface>`:

```bash
[root@pdai ~]# ethtool -S vethd93e2ad
NIC statistics:
     peer_ifindex: 50
[root@pdai ~]# docker exec -it 0ffd1092cd96 /bin/bash
root@0ffd1092cd96:/# ip a | grep 50
50: eth0@if51: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
root@0ffd1092cd96:/#

```

## [¶](#多物理机之间互联) 多物理机之间互联

> 如果在企业内部应用，或者做多个物理主机的集群，可能需要将多个物理主机的容器组到一个物理网络中来，那么就需要将这个网桥桥接到我们指定的网卡上。

### [¶](#拓扑图) 拓扑图

主机 A 和主机 B 的网卡一都连着物理交换机的同一个 vlan 101,这样网桥一和网桥三就相当于在同一个物理网络中了，而容器一、容器三、容器四也在同一物理网络中了，他们之间可以相互通信，而且可以跟同一 vlan 中的其他物理机器互联。

![img](E:\Development\Typora\images\docker-y-6.png)

### [¶](#ubuntu-示例) ubuntu 示例

下面以 ubuntu 为例创建多个主机的容器联网: 创建自己的网桥,编辑 /etc/network/interface 文件

```bash
auto br0
iface br0 inet static
address 192.168.7.31
netmask 255.255.240.0
gateway 192.168.7.254
bridge_ports em1
bridge_stp off
dns-nameservers 8.8.8.8 192.168.6.1
 
```



将 Docker 的默认网桥绑定到这个新建的 br0 上面，这样就将这台机器上容器绑定到 em1 这个网卡所对应的物理网络上了。

ubuntu 修改 /etc/default/docker 文件，添加最后一行内容

```bash
# Docker Upstart and SysVinit configuration file
# Customize location of Docker binary (especially for development testing).
#DOCKER="/usr/local/bin/docker"
# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"

# If you need Docker to use an HTTP proxy, it can also be specified here.
#export http_proxy="http://127.0.0.1:3128/"

# This is also a handy place to tweak where Docker's temporary files go.
#export TMPDIR="/mnt/bigdrive/docker-tmp"

DOCKER_OPTS="-b=br0"
  
        @pdai: 代码已经复制到剪贴板
    
```




在启动 Docker 的时候 使用 -b 参数 将容器绑定到物理网络上。重启 Docker 服务后，再进入容器可以看到它已经绑定到你的物理网络上了。

```bash
root@pdai:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                        NAMES
58b043aa05eb        desk_hz:v1          "/startup.sh"       5 days ago          Up 2 seconds        5900/tcp, 6080/tcp, 22/tcp   yanlx
root@pdai:~# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.7e6e617c8d53       no              em1
                                            vethe6e5
  
        @pdai: 代码已经复制到剪贴板
    
```



这样就直接把容器暴露到物理网络上了，多台物理主机的容器也可以相互联网了。需要注意的是，这样就需要自己来保证容器的网络安全了。

具体步骤，还可以看下这篇文章：[CentOS Docker跨宿主机通讯Open vSwitch](https://blog.csdn.net/gavinkelland/article/details/77976815)