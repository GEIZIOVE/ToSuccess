# 一、[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020) Compose 简介

`Compose` 项目是Docker官方的[开源项目](https://so.csdn.net/so/search?q=开源项目&spm=1001.2101.3001.7020)，负责实现对Docker容器集群的快速编排。从功能上看，跟[OpenStack](https://so.csdn.net/so/search?q=OpenStack&spm=1001.2101.3001.7020) 中的 Heat 十分类似。其代码目前在 https://github.com/docker/compose 上开源。

`Compose`定位是【定义和运行多个Docker容器的应用(Oefining and running mult-[container](https://so.csdn.net/so/search?q=container&spm=1001.2101.3001.7020) Docker applications)】，其前身是开源项目Fig。

我们知道使用一个Dockerfile模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个Web 项目，除了Web服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。Compose恰好满足了这样的需求。它允许用户通过一个单独的`docker-compose.yml`模板文件(YAML格式)来定义一组相关联的应用容器为一个项目
(project)。

Compose中有两个重要的概念:

- 服务 ( service ):一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目( project ):由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件中定义。

Compose的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。
Compose项目由Python编写，实现上调用了Docker 服务提供的API来对容器进行管理。因此，只要所操作的平台支持Docker API，就可以在其上利用Compose来进行编排管理。

compose以项目为核心，在一个项目中定义一组具有与项目相关联的（相同业务逻辑单元）的服务或容器

## 安装docker-compose

### 一、安装Docker Compose

```shell
# Compose目前已经完全支持Linux、Mac OS和Windows，在我们安装Compose之前，需要先安装Docker。下面我 们以编译好的二进制包方式安装在Linux系统中。 
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 设置文件可执行权限 
chmod +x /usr/local/bin/docker-compose
# 查看版本信息 
docker-compose -version
```

### 二、卸载Docker Compose

```shell
# 二进制包方式安装的，删除二进制文件即可
rm /usr/local/bin/docker-compose
```





```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose 
```

或者

```shell
yum install docker-compose # centos 版本较老
apt install docker-compose # ubuntu
```

安装成功执行

```shell
[root@localhost ~]# docker-compose -v
docker-compose version 1.29.2, build 8a1c60f6
```

安装bash命令补全

```shell
curl -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

## 快速上手一个项目

```c
[root@localhost ~]# mkdir docker-compose
[root@localhost ~]# cd docker-compose/

[root@localhost docker-compose]# cat docker-compose.yml
version: "3.0" # docker-compose模板的版本
services: # 服务,下面定义一组容器
  tomcat: # 具体的服务名
    image: tomcat:latest # 依赖的镜像
    ports: # 对外暴露的端口
      - 8080:8080


[root@localhost docker-compose]# docker-compose up  # 启动 -d 守护进程方式运行


[root@localhost ~]# docker ps
CONTAINER ID   IMAGE           COMMAND             CREATED          STATUS         PORTS                                       NAMES
712c42832e3e   tomcat:latest   "catalina.sh run"   11 seconds ago   Up 9 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   docker-compose_tomcat_1

```

# 二、Docker Compose模板指令

模板指令是写在docker-compose.yml文件中的，使用来为容器服务的。
指令是对整个项目操作的，格式：`docker-compose 指令`

## images

指定为镜像名称或镜像ID。如果镜像在本地不存在，Compose 将会尝试**拉取**这个镜像。

```c
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```

## ports

暴露端口信息。
使用宿主端口:容器端(HOST:CONTAINER）格式，或者仅仅指定容器的端(宿主将会**随机**选择端口)

```c
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

> <font color='red'>注意</font>：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 你可能会得到错误得结果，因为 YAML 将会解析 xx:yy 这种数字格式为 **60 进制**。所以建议采用字符串格式。

## volumes

卷挂载路径设置。可以设置宿主机路径 （`HOST:CONTAINER`） 或加上访问模（`HOST:CONTAINER:ro`）,挂载数据卷的默认权限是读写（rw），可以通过ro指定为只读。
你可以在主机上挂载相对路径，该路径将相对于当前正在使用的Compose配置文件的目录进行扩展。 相对路径应始终以 . 或者 … 开始。

```c
volumes:
  # 只需指定一个路径，让引擎创建一个卷
  - /var/lib/mysql
 
  # 指定绝对路径映射，需要事先创建绝对路径
  - /opt/data:/var/lib/mysql
 
  # 相对于当前compose文件的相对路径
  - ./cache:/tmp/cache
 
  # 用户根目录相对路径
  - ~/configs:/etc/configs/:ro
 
  # 命名卷，需要额外的声明，其创建的卷名为：项目名+datavolume，如果就需要使用自定义卷名，
  # 需要添加external，但是启动时事先需要先创建卷名，比较麻烦: docker volume create mydata
  - datavolume:/var/lib/mysql 
```

例如：

```shell
version: "3.0"
services:
  tomcat:
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      #- /apps:/usr/local/tomcat/webapps
      - tomcatwebapps:/usr/local/tomcat/webapps

volumes: # 声明上面的自动卷，如果是绝对路径或相对路径，不用声明
  tomcatwebapps:
    external：
      true # 确定使用自定义卷名，

启动时需要这样执行：
doclker volume create tomcatwebapps
docker-compose up
        

[root@localhost docker-compose]# cat docker-compose.yml 
version: "3.0"
services:
  tomcat:
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      #- /apps:/usr/local/tomcat/webapps
      - tomcatwebapps:/usr/local/tomcat/webapps  

volumes:    # 声明上面的自动卷
  tomcatwebapps:
    #external:
      #true   # 确定使用指定卷名，需要事先手动创建
      


# 查看卷，卷名为项目目录+tomcatwebapps
[root@localhost ~]# docker inspect docker-compose_tomcatwebapps
[
    {
        "CreatedAt": "2021-12-11T18:02:04+08:00",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "docker-compose",
            "com.docker.compose.version": "1.25.5",
            "com.docker.compose.volume": "tomcatwebapps"
        },
        "Mountpoint": "/var/lib/docker/volumes/docker-compose_tomcatwebapps/_data",
        "Name": "docker-compose_tomcatwebapps",
        "Options": null,
        "Scope": "local"
    }
]

```

## networks

配置容器连接网络

```c
version: "3"
services:
  some-service:
    networks:
      -some-network # 指定当前服务加入哪个网桥
      -other-network

networks:  # 创建网桥
  some-network:
  other-network:
  	external:
  	  true # 使用自定义网桥，但是必须事先创建:外部使用命令 docker network create some-network

```

## container_name

指定一个自定义容器名称，而不是生成的默认名称。

```c
container_name: my-web-container
```

由于Docker容器名称必须是唯一的，因此如果指定了自定义名称，则无法将服务扩展到多个容器。

例如，启动两个tomcat，并加入自建网络

```c
[root@localhost docker-compose]# cat docker-compose.yml 
version: "3.0"
services:
  tomcat1:
    container_name: tomcat01
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      - tomcatwebapps:/usr/local/tomcat/webapps  
    networks:
      - mynet

  tomcat2:
    container_name: tomcat02
    image: tomcat:latest
    ports:
      - "8081:8080" 
    volumes:
      - tomcatwebapps1:/usr/local/tomcat/webapps
    networks:
      - mynet

volumes:
  tomcatwebapps:
  tomcatwebapps1:

networks:
  mynet:
    external:
      true
  

```

运行

```c
[root@localhost docker-compose]# docker network create mynet
[root@localhost docker-compose]# docker-compose up
```

## environment

添加环境变量。 你可以使用数组或字典两种形式。

​	 **任何布尔值; true，false，yes，no需要用引号括起来，以确保它们不被YML解析器转换为True或False。**

只给定名称的变量会自动获取它在 `Compose` 主机上的值，可以用来防止泄露不必要的数据。
如果 `environment` 和`env_file`同时存在，前者会覆盖后者

```yml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
 
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

注意：如果你的服务指定了build选项，那么在构建过程中通过`environment`定义的环境变量将不会起作用。 将使用build的args子选项来定义构建时的环境变量。

例如mysql服务

```c
[root@localhost docker-compose]# cat docker-compose.yml 
version: "3.0"
services:
  tomcat1:
    container_name: tomcat01
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      - tomcatwebapps:/usr/local/tomcat/webapps  
    networks:
      - mynet

  tomcat2:
    container_name: tomcat02
    image: tomcat:latest
    ports:
      - "8081:8080" 
    volumes:
      - tomcatwebapps1:/usr/local/tomcat/webapps
    networks:
      - mynet

  mysql:
    container_name: mysql01
    image: mysql:5.7
    ports:
      - "3306:3306"
    
    volumes:
      - /data/mysql:/var/lib/mysql
      - mysqlconf:/ect/mysql
    networks:
      - mynet
    environment:
      - MYSQL_ROOT_PASSWORD=123456

volumes:
  tomcatwebapps:
  tomcatwebapps1:
  mysqlconf:

networks:
  mynet:
    external:
      true
 
```

## command:

**使用 command 可以覆盖容器启动后默认执行的命令。**

```c
command: bundle exec thin -p 3000

command: ["bundle", "exec", "thin", "-p", "3000"]
```

例如redis服务

```c
[root@localhost docker-compose]# cat docker-compose.yml 
version: "3.0"
services:
  tomcat1:
    container_name: tomcat01
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      - tomcatwebapps:/usr/local/tomcat/webapps  
    networks:
      - mynet

  tomcat2:
    container_name: tomcat02
    image: tomcat:latest
    ports:
      - "8081:8080" 
    volumes:
      - tomcatwebapps1:/usr/local/tomcat/webapps
    networks:
      - mynet

  mysql:
    container_name: mysql01
    image: mysql:5.7
    ports:
      - "3306:3306"
    volumes:
      - /data/mysql:/var/lib/mysql
      - mysqlconf:/ect/mysql
    networks:
      - mynet
    environment:
      - MYSQL_ROOT_PASSWORD=123456
  redis:
    container_name: redis01
    image: redis:5.0.10
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    networks:
      - mynet
    command: "redis-server --appendonly yes" # 覆盖默认启动命令 开启redis数据持久化

volumes:
  tomcatwebapps:
  tomcatwebapps1:
  mysqlconf:
  redisdata:

networks:
  mynet:
    external:
      true
```

## env_file

从一个文件中加入环境变量，带入到容器中去，在容器中可以用printenv打印该环境变量。该文件可以是一个单独的值或者一张列表，**在`environment`中指定的环境变量将会重写这些值**

```yml
env_file : .env
或
env_file:
 - ./common.env
 - ./apps/ web.env
 - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持#开头的注释行。

```yml
# common.env: Set development environment
MYSQL_ROOT_PASSWORD = "123456"
```

## depends_on

解决容器的依赖、启动先后的问题。以下例子中会先启动redis ,db再启动web `此处是服务名，而不是容器名`

```YAML
version: "3.8"
services:
  web:
    build: .
    depends_on:
      - db # 注意此处是服务名，而不是容器名
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

注意: web服务不会等待redis db「完全启动」之后才启动。
例如 tomcat服务，需要依赖于mysql 和redis

```c
[root@localhost docker-compose]# cat docker-compose.yml
version: "3.0"
services:
  tomcat1:
    container_name: tomcat01
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      - tomcatwebapps:/usr/local/tomcat/webapps  
    networks:
      - mynet
    depends_on:
      - mysql
      - redis

  mysql:
    container_name: mysql01
    image: mysql:5.7
    ports:
      - "3306:3306"
    volumes:
      - /data/mysql:/var/lib/mysql
      - mysqlconf:/ect/mysql
    networks:
      - mynet
    environment:
      - MYSQL_ROOT_PASSWORD=123456
  
  redis:
    container_name: redis01
    image: redis:5.0.10
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    networks:
      - mynet
    command: "redis-server --appendonly yes"
volumes:
  tomcatwebapps:
  mysqlconf:
  redisdata:

networks:
  mynet:
    external:
      true
```

## healthcheck

通过命令检查容器是否健康运行。需要给每个service都添加

```c
healthcheck:
  test: [ "CMD", "curl", "-f", “http://localhost"]
  interval: 1m30s # 前多少秒不检查
  timeout: 10s # 等待时间
  sretries: 3 # 尝试三次
```

## sysctls

配置容器内核参数，下面两种写法都可以，并不是必须的，有些服务受容器内操作系统内核参数的限制，可能无法启动，必须通过修改内核参数才能启动

```c
sysctls :
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls :
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

## ulimits

指定容器的ulimits限制值。
例如，指定最大进程数为65535，指定文件句柄数为200 (软限制，应用可以随时修改，不能超过硬限制)和4000(系统硬限制，只能root 用户提高）

```c
ulimits:
  nproc: 65535 # 修改容器内操作系统的最大进程数，可根据服务的要求修改
  nofile:
    soft: 2000
    hard: 4000
```

## build指令

`用来将指定的Dockerfile打包成对应镜像，然后再运行该镜像，不需要手动先构建镜像`

```c
services:
  web:
    build:  
      context:  /app/demo# 指定上下文目录,可以写绝对路径也可以写相对路径,建议将Dockerfile的目录和Docker-compose.yml文件放在同一目录，这样就可以使用相对路径了
      dockerfile: Dockerfile # 文件名
    container_name: demo
    networks:
      - mynet
    ....
```

## ARGS

添加构建镜像的参数，环境变量只能在构建过程中访问。
首先，在Dockerfile中指定要使用的参数：

```c
ARG buildno
ARG password
 
RUN echo "Build number: $buildno"
RUN script-requiring-password.sh "$password"
```

然后在args键下指定参数。 你可以传递映射或列表：

```c
build:
  context: .
  args:
    buildno: 1
    password: secret
 
build:
  context: .
  args:
    - buildno=1
    - password=secret
```

# 三、Docker Compose常用命令

注： 此命令后跟的都是服务名或服务id，而不是容器名
执行 `docker-compose [COMMAND] --help` 或者 `docker-compose help [COMMAND]`x可以查看具体某个命令的使用格式。
docker-compose 命令的基本的使用格式是

```c
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

命令选项

```c
-f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。

-p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。

--verbose 输出更多调试信息。

-v, --version 打印版本并退出。
```

## up

- 该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。
- 链接的服务都将会被自动启动，除非已经处于运行状态。
- **可以说，大部分时候都可以直接通过该命令来启动一个项目。**
- 默认情况，docker-compose up 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。
- 当通过 Ctrl-C 停止命令时，所有容器将会停止。
- **如果使用 docker-compose up -d，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。**
- 默认情况，如果服务容器已经存在，docker-compose up 将会尝试停止容器，然后重新创建（保持使用 volumes-from 挂载的卷），以保证新启动的服务匹配 docker-compose.yml 文件的最新内容。如果用户不希望容器被停止并重新创建，可以使用 `docker-compose up --no-recreate`。这样将只会启动处于停止状态的容器，而忽略已经运行的服务。如果用户只想重新部署某个服务，可以使用 `docker-compose up --no-deps -d <SERVICE_NAME>` 来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。

选项：

```c
-d 在后台运行服务容器。

--no-color 不使用颜色来区分不同的服务的控制台输出。

--no-deps 不启动服务所链接的容器。

--force-recreate 强制重新创建容器，不能与 --no-recreate 同时使用。

--no-recreate 如果容器已经存在了，则不重新创建，不能与 --force-recreate 同时使用。

--no-build 不自动构建缺失的服务镜像。

-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
```

## config

验证 Compose 文件语法格式是否正确，若正确则显示配置，若格式错误显示错误原因。

## down

此命令将会停止 up 命令所启动的容器，并移除网络

注：以下命令全都跟SERVIC(服务)

## exec

`docker-compose exec [SERVICE] bash`
进入指定的容器。

```c
docker-compose exec 服务id（不是容器名） bash
```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiHd3Xnmoblj6_niLE=,size_20,color_FFFFFF,t_70,g_se,x_16.png)
进入容器：

```yaml
docker-compose exec mysqldb bash
```

## ps

格式为 `docker-compose ps [options] [SERVICE...]`。

列出项目中目前的所有容器。

选项：

-q 只打印容器的 ID 信息

```c
[root@localhost docker-compose]# docker-compose ps
  Name                Command               State                          Ports                       
-------------------------------------------------------------------------------------------------------
mysql01    docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp,:::3306->3306/tcp, 33060/tcp
redis01    docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp,:::6379->6379/tcp           
tomcat01   catalina.sh run                  Up      0.0.0.0:8080->8080/tcp,:::8080->8080/tcp           

1234567
```

## restart

格式为 `docker-compose restart [options] [SERVICE...]`。

重启项目中的服务。如果不写service id,默认重启所有服务

选项：

```c
-t, --timeout TIMEOUT 指定重启前停止容器的超时（默认为 10 秒）
1
```

## rm

格式为 `docker-compose rm [options] [SERVICE...]`。

删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器。

选项：

```c
-f, --force 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。

-v 删除容器所挂载的数据卷
123
```

## start

格式为 `docker-compose start [SERVICE...]`。

启动已经存在的服务容器。

## stop

格式为 `docker-compose stop [options] [SERVICE...]`。

停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器。

选项：

```c
-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
```

## top

查看各个服务容器内运行的进程。
docker-compose top [options] [SERVICE…]

```c
[root@localhost docker-compose]# docker-compose top mysql
mysql01
  UID     PID    PPID   C   STIME   TTY     TIME      CMD  
-----------------------------------------------------------
polkitd   2325   2300   0   15:26   ?     00:00:00   mysqld
[root@localhost docker-compose]# 
[root@localhost docker-compose]# 
[root@localhost docker-compose]# 
[root@localhost docker-compose]# docker-compose top redis
redis01
  UID     PID    PPID   C   STIME   TTY     TIME             CMD        
------------------------------------------------------------------------
polkitd   2289   2270   0   15:26   ?     00:00:00   redis-server *:6379
[root@localhost docker-compose]# 
```

## logs

格式为 `docker-compose logs [options] [SERVICE...]`。

查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 --no-color 来关闭颜色。
该命令在调试问题的时候十分有用

更多命令请参考[命令大全](https://vuepress.mirror.docker-practice.com/compose/commands)

# 四、Docker 可视化工具 Portaine使用

[官方安装说明:](https://www.portainer.io/installation/)

```c
docker pull portainer/portainer
docker volume create portainer_data

docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data/data portainer/portainer
1234
```

`porttainer`服务需要与`docker`引擎通讯，通过sock文件，这样才能检测到具体的容器。端口为8000，其中9000端口供外部访问
其中portainer_data为portainer容器保存的数据
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiHd3Xnmoblj6_niLE=,size_20,color_FFFFFF,t_70,g_se,x_16-16640431023881.png)

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiHd3Xnmoblj6_niLE=,size_20,color_FFFFFF,t_70,g_se,x_16-16640431023902.png)
生产环境中，不建议用docker命令的方式启动portainer容器，需要与docker-compose项目一起启动

```c
 portainer:
    container_name: portainer
    image: portainer/portainer
    ports:
      - "8000:8000"
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data/data portainer/portainer
    networks:
      mynet

volumes:
  portainer_data:
    external: true

networks:
  mynet:
    external: true    
```

启动：

```c
docker network create mynet
docker volume create portain_data
docker-compose up
```