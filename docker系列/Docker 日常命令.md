# Docker 日常命令

### [Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)命令及用法

[命令大全](https://www.runoob.com/docker/docker-command-manual.html)

| Docker 命令        | 命令说明                                      | 具体用法                                                     |
| :----------------- | :-------------------------------------------- | :----------------------------------------------------------- |
| **docker run**     | 创建一个新的容器并运行一个命令                | [具体用法](https://www.runoob.com/docker/docker-run-command.html) |
| **docker start**   | 启动一个或多个已经被停止的容器                | docker start 容器名称 / 容器ID                               |
| **docker stop**    | 停止一个运行中的容器                          | docker stop 容器名称 / 容器ID                                |
| **docker restart** | 重启容器                                      | docker restart 容器名称 / 容器ID                             |
| **docker kill**    | 杀掉一个运行中的容器                          | docker kill -s KILL 容器名称 / 容器ID                        |
| **docker rm**      | 删除一个或多个容器                            | docker rm -f 容器名称 / 容器ID                               |
| **docker pause**   | 暂停容器中所有的进程                          | docker pause 容器名称 / 容器ID                               |
| **docker unpause** | 恢复容器中所有的进程                          | docker unpause 容器名称 / 容器ID                             |
| **docker create**  | 创建一个新的容器但不启动它                    | [具体用法](https://www.runoob.com/docker/docker-create-command.html) |
| **docker exec**    | 在运行的容器中执行命令                        | [具体用法](https://www.runoob.com/docker/docker-exec-command.html) |
| **docker ps**      | 列出容器                                      | [具体用法](https://www.runoob.com/docker/docker-ps-command.html) |
| **docker logs**    | 获取容器的日志                                | [具体用法](https://www.runoob.com/docker/docker-logs-command.html) |
| **docker login**   | 登陆Docker镜像仓库，默认为官方仓库 Docker Hub | [具体用法](https://www.runoob.com/docker/docker-login-command.html) |
| **docker logout**  | 登出Docker镜像仓库，默认为官方仓库 Docker Hub | docker logout                                                |
| **docker pull**    | 从镜像仓库中拉取或者更新指定镜像              | [具体用法](https://www.runoob.com/docker/docker-pull-command.html) |
| **docker search**  | 从Docker Hub查找镜像                          | docker search 容器名称                                       |
| **docker images**  | 列出本地镜像                                  | [具体用法](https://www.runoob.com/docker/docker-images-command.html) |
| **docker build**   | 命令用于使用 Dockerfile 创建镜像              | [具体用法](https://www.runoob.com/docker/docker-build-command.html) |
| docker info        | 显示 Docker 系统信息，包括镜像和容器数        | docker info                                                  |
| docker version     | 显示 Docker 版本信息                          | docker version                                               |

### 日常使用到的命令

```bash
## 查看本地镜像
docker images
 
## 查看运行中的镜像
docker ps 
 
## 查看所有镜像,包括未运行的
docker ps -a
 
## 启动某个镜像
docker start mysql
 
## 关闭某个镜像
docker stop mysql
 
## 重启某个镜像
docker restart mysql
 
## 强制关闭运行中的容器
docker kill -s KILL mysql
 
## 进入某个容器内部 (如 : mysql)
docker exec -it mysql /bin/bash

```



docker run [OPTIONS] IMAGE [COMMAND] [ARG…]

> OPTIONS参数说明：
> -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
> -d: 后台运行容器，并返回容器ID；
> -i: 以交互模式运行容器，通常与 -t 同时使用；
> -P: 随机端口映射，容器内部端口随机映射到主机的端口
> -p: 指定端口映射，格式为：主机（宿主）端口:容器端口
> -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
> name=“nginx-lb”: 为容器指定一个名称；
> dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
> dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
> -h “mars”: 指定容器的hostname；
> -e username=“ritchie”: 设置环境变量；
> env-file=[]: 从指定文件读入环境变量；
> cpuset=“0-2” or cpuset=“0,1,2”: 绑定容器到指定CPU运行；
> -m :设置容器使用内存最大值；
> net=“bridge”: 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
> link=[]: 添加链接到另一个容器；
> expose=[]: 开放一个端口或一组端口；
> volume , -v: 绑定一个卷
> 例如，使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx：
> docker run name mynginx -d nginx:latest
> 使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口：
> docker run -P -d nginx:latest
> 使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data：
> docker run -p 80:80 -v /data:/data -d nginx:latest
> 绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上：
> docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
> 使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令：
> docker run -it nginx:latest /bin/bash