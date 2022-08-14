### Linux环境安装[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)环境

- - [安装前提条件](https://blog.csdn.net/weixin_43865008/article/details/121916395#_2)
  - [第一步：检查并清除系统残余项，并安装Docker依赖环境](https://blog.csdn.net/weixin_43865008/article/details/121916395#Docker_12)
  - [第二步：Docker依赖环境搭建好之后，安装并启动Docker](https://blog.csdn.net/weixin_43865008/article/details/121916395#DockerDocker_113)



## 安装前提条件

Docker 要求 CentOS 系统的内核版本高于 3.10 ，首先验证你的服务器是否支持Docker！

通过 `uname -r` 命令查看当前的内核版本

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# uname -r
4.18.0-193.14.2.el8_2.x86_64
```

可以看到我的服务器是4.18.0，是支持Docker的。

## 第一步：检查并清除系统残余项，并安装Docker依赖环境

1、清除残余

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# sudo yum remove docker \
                           docker-client \
                           docker-client-latest \
                           docker-common \
                           docker-latest \
                           docker-latest-logrotate \
                           docker-logrotate \
                           docker-selinux \
                           docker-engine-selinux \
                           docker-engine
```

执行结果

```bash
No match for argument: docker
No match for argument: docker-client
No match for argument: docker-client-latest
No match for argument: docker-common
No match for argument: docker-latest
No match for argument: docker-latest-logrotate
No match for argument: docker-logrotate
No match for argument: docker-selinux
No match for argument: docker-engine-selinux
No match for argument: docker-engine
没有软件包需要移除。
依赖关系解决。
无需任何处理。
完毕！
```

我这台是新租的服务器的，所以没有依赖项存在。（为了保障docker的顺利安装还是执行了一下）

**安装下载Docker依赖的工具**

```cpp
[root@iZbp18425116ezmjdmbdgeZ ~]# sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

结果

```csharp
已升级:
  device-mapper-8:1.02.177-10.el8.x86_64                                  device-mapper-event-8:1.02.177-10.el8.x86_64                      
  device-mapper-event-libs-8:1.02.177-10.el8.x86_64                       device-mapper-libs-8:1.02.177-10.el8.x86_64                       
  device-mapper-persistent-data-0.9.0-4.el8.x86_64                        dnf-4.7.0-4.el8.noarch                                            
  dnf-data-4.7.0-4.el8.noarch                                             dnf-plugins-core-4.0.21-3.el8.noarch                              
  ima-evm-utils-1.3.2-12.el8.x86_64                                       libdnf-0.63.0-3.el8.x86_64                                        
  librepo-1.14.0-2.el8.x86_64                                             libsolv-0.7.19-1.el8.x86_64                                       
  lvm2-8:2.03.12-10.el8.x86_64                                            lvm2-libs-8:2.03.12-10.el8.x86_64                                 
  python3-dnf-4.7.0-4.el8.noarch                                          python3-dnf-plugins-core-4.0.21-3.el8.noarch                      
  python3-hawkey-0.63.0-3.el8.x86_64                                      python3-libdnf-0.63.0-3.el8.x86_64                                
  python3-librepo-1.14.0-2.el8.x86_64                                     python3-rpm-4.14.3-19.el8.x86_64                                  
  rpm-4.14.3-19.el8.x86_64                                                rpm-build-libs-4.14.3-19.el8.x86_64                               
  rpm-libs-4.14.3-19.el8.x86_64                                           rpm-plugin-selinux-4.14.3-19.el8.x86_64                           
  rpm-plugin-systemd-inhibit-4.14.3-19.el8.x86_64                         yum-4.7.0-4.el8.noarch                                            

已安装:
  libmodulemd-2.13.0-1.el8.x86_64                 tpm2-tss-2.3.2-4.el8.x86_64                 yum-utils-4.0.21-3.el8.noarch                

完毕！
```

这里内容比较长，我只截取了结尾部分

**添加阿里云的软件源**

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

结果：

```csharp
Loaded plugins: fastestmirror
adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```

以后每个软件都优先从阿里云的软件库中下载，如果阿里云仓库没有，会去docker.hub中下载。（与maven仓库同理）

**更新yum缓存（为了保证能更新和下载需要的服务：如docker）**

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# sudo yum makecache 
```

结果：

```bash
[root@iZbp18425116ezmjdmbdgeZ ~]# sudo yum makecache
Invalid configuration value: failovermethod=priority in /etc/yum.repos.d/CentOS-epel.repo; 配置：ID 为 "failovermethod" 的 OptionBinding 不存在
CentOS-8 - AppStream                                                                                        441 kB/s | 4.3 kB     00:00    
CentOS-8 - Base                                                                                             437 kB/s | 3.9 kB     00:00    
CentOS-8 - Extras                                                                                           194 kB/s | 1.5 kB     00:00    
Extra Packages for Enterprise Linux 8 - x86_64                                                              602 kB/s | 4.7 kB     00:00    
Docker CE Stable - x86_64                                                                                    30 kB/s |  19 kB     00:00    
元数据缓存已建立。
```

## 第二步：Docker依赖环境搭建好之后，安装并启动Docker

**1、安装Docker（CE-社区版）**

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# sudo yum -y install docker-ce
```

由于需要下载并安装docker，有的机器会有点慢，请耐心等待！

结果：

```bash
Installed:
  docker-ce.x86_64 3:18.09.0-3.el7
 
Dependency Installed:
  audit-libs-python.x86_64 0:2.8.1-3.el7_5.1      checkpolicy.x86_64 0:2.5-6.el7              container-selinux.noarch 2:2.68-1.el7
  containerd.io.x86_64 0:1.2.0-3.el7              docker-ce-cli.x86_64 1:18.09.0-3.el7        libcgroup.x86_64 0:0.41-15.el7
  libseccomp.x86_64 0:2.3.1-3.el7                 libsemanage-python.x86_64 0:2.5-11.el7      libtool-ltdl.x86_64 0:2.4.2-22.el7_3
  policycoreutils-python.x86_64 0:2.5-22.el7      python-IPy.noarch 0:0.75-6.el7              setools-libs.x86_64 0:3.3.8-2.el7
 
Dependency Updated:
  audit.x86_64 0:2.8.1-3.el7_5.1                       audit-libs.x86_64 0:2.8.1-3.el7_5.1   libselinux.x86_64 0:2.5-12.el7
  libselinux-python.x86_64 0:2.5-12.el7                libselinux-utils.x86_64 0:2.5-12.el7  libsemanage.x86_64 0:2.5-11.el7
  libsepol.x86_64 0:2.5-8.1.el7                        policycoreutils.x86_64 0:2.5-22.el7   selinux-policy.noarch 0:3.13.1-192.el7_5.6
  selinux-policy-targeted.noarch 0:3.13.1-192.el7_5.6
 
Complete!

```

看到complete！下载并安装成功！

**2、启动Docker服务**

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]#  sudo systemctl start docker
```

以上我们已经将Docker安装好了，接下来测试下Docker是否可以顺利启动：

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# docker info
```

结果：

```bash
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.6.3-docker)
  scan: Docker Scan (Docker Inc., v0.9.0)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.11
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
```

我们可以看到Docker已经启动成功，而且容器与镜像数都为0，是一个全新的docker服务

**3、设置开机自启（非必设项，根据自己习惯设置）**

```bash
[root@iZx4xwfjh1zsdsZ /]# sudo systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@iZx4xwfjh1zsdsZ /]#
```

到此为止，Docker就已经下载并安装完成！

**查看docker版本**

```csharp
[root@iZbp18425116ezmjdmbdgeZ ~]# docker -v
Docker version 20.10.11, build dea9396
```

**移除Docker-ce服务**

```bash
sudo yum remove docker-ce
```

**删除Docker依赖项**

```csharp
sudo rm -rf /var/lib/docker
```