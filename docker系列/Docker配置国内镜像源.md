# Docker配置国内镜像源

[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)在默认安装之后，通过命令docker pull 拉取镜像时，默认访问docker hub上的镜像，在国内网络环境下，下载时间较久，所以要配置国内镜像仓库。

修改方式如下：

```bash
第一步：新建或编辑daemon.json
vi /etc/docker/daemon.json
 
第二步：daemon.json中编辑如下
{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
 
第三步：重启docker
systemctl restart docker.service
 
第四步：执行docker info查看是否修改成功
docker info
```

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvdXBrOTk5,size_16,color_FFFFFF,t_70.png)

国内的加速地址如下：

```javascript
1.网易
http://hub-mirror.c.163.com
2.Docker中国区官方镜像
https://registry.docker-cn.com
3.中国科技大学
https://docker.mirrors.ustc.edu.cn
4.阿里云容器  服务
https://cr.console.aliyun.com/
首页点击“创建我的容器镜像”  得到一个专属的镜像加速地址，类似于“https://1234abcd.mirror.aliyuncs.com”
```