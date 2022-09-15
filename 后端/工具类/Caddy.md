# Caddy简介

[Caddy中文文档 - Caddy中文文档 (dengxiaolong.com)](https://dengxiaolong.com/caddy/zh/)

Caddy是一款功能强大，扩展性高的Web服务器，目前在Github上已有`38K+Star`。Caddy采用Go语言编写，可用于静态资源托管和反向代理。

![img](E:\Development\Typora\images\caddy_start_15.65638303.png)

Caddy具有如下主要特性：

- 对比Nginx复杂的配置，其独创的`Caddyfile`配置非常简单；
- 可以通过其提供的`Admin API`实现动态修改配置；
- 默认支持自动化HTTPS配置，能自动申请HTTPS证书并进行配置；
- 能够扩展到数以万计的站点；
- 可以在任意地方执行，没有额外的依赖；
- 采用Go语言编写，内存安全更有保证。

# 一、安装



## 1.直接安装

> 首先我们直接在CentOS 8上安装Caddy，使用DNF工具安装无疑是最简单的，Docker安装方式之后也会介绍。

- 使用如下命令通过DNF工具安装Caddy，安装成功后Caddy会被注册成系统服务；

```bash
dnf install 'dnf-command(copr)'
dnf copr enable @caddy/caddy
dnf install caddy
```

- 使用`systemctl status caddy`查看Caddy的状态，可以发现Caddy已被注册为系统服务，但是还没开启。

![img](E:\Development\Typora\images\caddy_start_01.274987ed.png)

## 2.Docker支持

> 当然Caddy也是支持使用Docker进行安装使用的，其使用和直接在CentOS上安装基本一致。

- 首先使用如下命令下载Caddy的Docker镜像；

```bash
docker pull caddy
```

- 然后在`/mydata/caddy/`目录下创建`Caddyfile`配置文件，文件内容如下；

```text
http://192.168.3.105:80

respond "Hello, world!"
```

- 之后使用如下命令启动caddy服务，这里将宿主机上的`Caddyfile`配置文件、Caddy的数据目录和网站目录挂载到了容器中；

```bash
docker run -p 80:80 -p 443:443 --name caddy \
    -v /mydata/caddy/Caddyfile:/etc/caddy/Caddyfile \
    -v /mydata/caddy/data:/data \
    -v /mydata/caddy/html:/usr/share/caddy \
    -d caddy
```

- 之后使用`docker exec`进入caddy容器内部执行命令；

```bash
docker exec -it caddy /bin/sh
```

- 输入Caddy命令即可操作，之后的操作就和我们直接在CentOS上安装一样了。

![img](E:\Development\Typora\images\caddy_start_14.1dac30f7.png)



# 二、快速入门

## Hello，World！

> 首先我们来个Caddy的入门使用，让Caddy运行在`2015`端口上并返回`Hello, world!`。

- 直接使用`caddy`命令将输出Caddy的常用命令，基本看介绍就知道如何使用了，标出来的是常用命令；

![img](E:\Development\Typora\images\caddy_start_02.43eea14e.png)

- 使用`caddy start`命令可以让Caddy服务在后台运行；

![img](E:\Development\Typora\images\caddy_start_03.cdf118d3.png)

- Caddy默认使用JSON格式的配置文件，但由于JOSN格式配置书写比较麻烦，又提供了`Caddyfile`这种更加简洁的配置形式，使用如下命令能自动把`Caddyfile`转化为JSON配置；

```bash
caddy adapter
```

- 我们可以先创建一个名称为`Caddyfile`的文件，文件内容如下，然后使用`caddy adapter`将它转换为JSON配置，再使用`caddy reload`使配置生效，该配置将监听`2015`端口，并返回`Hello, world!`；

> 提示
>
> 创建一个名为Caddyfile（无扩展名）的新文本文件。
>
> 最先在Caddyfile输入的内容是你的站点访问地址：
>
> `Caddyfile`文件在自己的文件夹创建就好，然后再Caddyfile所在目录运行命令就ok
>
> ![image-20220908123124332](E:\Development\Typora\images\image-20220908123124332.png)

```text
:2015

respond "Hello, world!"
```

- 然后我们使用curl命令访问`localhost:2015`，将返回指定的信息；

![img](E:\Development\Typora\images\caddy_start_04.a850166a.png)

- 当前JSON配置如下，如果你直接使用JSON配置的话需要书写如下配置，使用`Caddyfile`确实方便很多！



```json
{
	"apps": {
		"http": {
			"servers": {
				"srv0": {
					"listen": [":2015"],
					"routes": [{
						"handle": [{
							"body": "Hello, world!",
							"handler": "static_response"
						}]
					}]
				}
			}
		}
	}
}
```

## `Caddyfile`基本语法

- 下面案例将使用`Caddyfile`来进行配置，我们有必要了解下它的语法，`Caddyfile`的具体语法规则如下。

![img](E:\Development\Typora\images\caddy_start_05.c7b34570.png)

- 介绍下上图中的关键字，有助于理解。

| 关键字               | 解释               | 使用                                                         |
| -------------------- | ------------------ | ------------------------------------------------------------ |
| Global options block | 服务器全局配置     | 可用于配置是否启用HTTPS和Admin API等                         |
| Snippet              | 可以复用的配置片段 | 定义好后认可以通过`import`关键字引用                         |
| Site Block           | 单个网站配置       | 通过`file_server`可以配置静态代理，通过`reverse_proxy`可以配置动态代理 |
| Matcher definition   | 匹配定义           | 默认情况下指令会产生全局影响，通过它可以指定影响范围         |
| Comment              | 注释               | 使用`#`符号开头                                              |
| Site address         | 网站地址           | 默认使用HTTPS，如需开启HTTP，需要指定`http://`开头           |
| Directive            | 指令               | 指令赋予了Caddy强大的功能                                    |

## 反向代理

> 反向代理就是当请求访问你的代理服务器时，代理服务器会对你的请求进行转发，可以转发到静态的资源路径上去，也可以转发到动态的服务接口上去。下面我们以对域名进行代理为例，来讲讲如何进行静态代理和动态代理。

### 静态代理

> 静态代理就是将请求代理到不同的**静态资源路径**上去，这里我们将对`docs.macrozheng.com`的请求代理到我的文档项目中，对`mall.macrozheng.com`的请求代理到mall的前端项目中。

- 首先我们修改下本机的host文件：

```text
192.168.3.106 docs.macrozheng.com
192.168.3.106 mall.macrozheng.com
```

- 然后将我们的文档项目和mall前端项目上传到Caddy的html目录中去，并进行解压操作：

![img](E:\Development\Typora\images\caddy_start_06.72d2e6b1.png)

- 修改`Caddyfile`文件，使用如下配置，修改完成后使用`caddy reload`命令刷新配置；

```text
http://docs.macrozheng.com {
        root * /mydata/caddy/html/docs
        file_server browse
}

http://mall.macrozheng.com {
        root * /mydata/caddy/html/mall
        file_server browse
}
```

- 如果你的`Caddyfile`文件格式不太合格的话，会出现如下警告，直接使用`caddy fmt --overwrite`格式化并重写配置即可解决；

![img](E:\Development\Typora\images\caddy_start_07.240a5d4f.png)

- 通过`docs.macrozheng.com`即可访问部署好的文档项目了：

![img](E:\Development\Typora\images\caddy_start_08.d8e61b3f.png)

- 通过`mall.macrozheng.com`即可访问到部署好的前端项目了。

![img](E:\Development\Typora\images\caddy_start_09.2ad09442.png)

### 动态代理

> 动态代理就是把代理服务器的请求转发到另一个服务上去，这里我们将把对`api.macrozheng.com`的请求代理到演示环境的API服务上去。

- 首先我们修改下本机的host文件，添加如下规则：

```bash
192.168.3.106 api.macrozheng.com
```

- 修改`Caddyfile`文件，使用如下配置，修改完成后使用`caddy reload`命令刷新配置；

```text
http://api.macrozheng.com {
        reverse_proxy http://admin-api.macrozheng.com
}
```

- 之后通过`api.macrozheng.com/swagger-ui.html`即可访问到`mall-admin`的API文档页面了。

![img](E:\Development\Typora\images\caddy_start_10.5e8513df.png)

### 文件压缩

> 如果我们的服务器带宽比较低，网站访问速度会很慢，这时我们可以通过让Caddy开启Gzip压缩来提高网站的访问速度。这里我们以mall的前端项目为例来演示下它的提速效果。

- 我们需要修改`Caddyfile`文件，使用`encode`指令开启Gzip压缩，修改完成后使用`caddy reload`命令刷新配置；



```text
http://mall.macrozheng.com {
        root * /mydata/caddy/html/mall
        encode {
            gzip
        }
        file_server browse
}
```

- 有个比较大的JS文件压缩前是`1.7M`；

![img](E:\Development\Typora\images\caddy_start_11.d2c76c3a.png)

- 压缩后为`544K`，访问速度也有很大提示；

![img](E:\Development\Typora\images\caddy_start_12.eba5db6c.png)

- 另外我们可以看下响应信息，如果有`Content-Encoding: gzip`这个响应头表明Gzip压缩已经启用了。

![img](E:\Development\Typora\images\caddy_start_13.4deb747b.png)

### 地址重写

> 有的时候我们的网站更换了域名，但还有用户在使用老的域名访问，这时可以通过Caddy的地址重写功能来让用户跳转到新的域名进行访问。

- 我们需要修改`Caddyfile`文件，使用`redir`指令重写地址，修改完成后使用`caddy reload`命令刷新配置；

```text
http://docs.macrozheng.com {
        redir http://www.macrozheng.com
}
```

- 此时访问旧域名`docs.macrozheng.com`会直接跳转到`www.macrozheng.com`去。

### 按目录划分

> 有时候我们需要使用同一个域名来访问不同的前端项目，这时候就需要通过子目录来区分前端项目了。

- 比如说我们需要按以下路径来访问各个前端项目；

```text
www.macrozheng.com #访问文档项目
www.macrozheng.com/admin #访问后台项目
www.macrozheng.com/app #访问移动端项目
```

- 我们需要修改`Caddyfile`文件，使用`route`指令定义路由，修改完成后使用`caddy reload`命令刷新配置。

```text
http://www.macrozheng.com {
        route /admin/* {
                uri strip_prefix /admin
                file_server {
                        root /mydata/caddy/html/admin
                }
        }
        route /app/* {
                uri strip_prefix /app
                file_server {
                        root /mydata/caddy/html/app
                }
        }
        file_server * {
                root /mydata/caddy/html/www
        }
}
```

### HTTPS

> Caddy能自动支持HTTPS，无需手动配置证书，这就是之前我们在配置域名时需要使用`http://`开头的原因，要想使用Caddy默认的HTTPS功能，按如下步骤操作即可。

- 首先我们需要修改域名的DNS解析，直接在购买域名的网站上设置即可，这里以`docs.macrozheng.com`域名为例；
- 之后使用如下命令验证DNS解析记录是否正确，注意配置的服务器的`80`和`443`端口需要在外网能正常访问；

```bash
curl "https://cloudflare-dns.com/dns-query?name=docs.macrozheng.com&type=A" \
  -H "accept: application/dns-json"
```

- 修改`Caddyfile`配置文件，进行如下配置；

```text
docs.macrozheng.com {
        root * /mydata/caddy/html/docs
        file_server browse
}
```

- 然后使用`caddy run`命令启动Caddy服务器即可，是不是非常方便！

```bash
caddy run
```



# 三、参考

## 1.命令行

Caddy有一个标准的类unix命令行接口，基本用法如下：

```
caddy <command> [<args...>]
```

- `<>`代表要被你输入替换参数。
- `[]`代表可选的参数。
- `...`表示延续，即一个或多个参数。

**快速开始：`caddy help`**

- [caddy adapt](https://caddy2.dengxiaolong.com/docs/command-line#caddy-adapt) 将配置文档适配为原生JSON
- [caddy build-info](https://caddy2.dengxiaolong.com/docs/command-line#caddy-build-info) 打印构建信息
- [caddy environ](https://caddy2.dengxiaolong.com/docs/command-line#caddy-environ) 打印环境
- [caddy file-server](https://caddy2.dengxiaolong.com/docs/command-line#caddy-file-server) 一个简单但可用于生产的文件服务器
- [caddy fmt](https://caddy2.dengxiaolong.com/docs/command-line#caddy-fmt) 格式化一个 Caddyfile
- [caddy hash-password](https://caddy2.dengxiaolong.com/docs/command-line#caddy-hash-password) 散列密码并输出 base64
- [caddy help](https://caddy2.dengxiaolong.com/docs/command-line#caddy-help) 查看 caddy 命令的帮助
- [caddy list-modules](https://caddy2.dengxiaolong.com/docs/command-line#caddy-list-modules) 列出已安装的 Caddy 模块
- [caddy reload](https://caddy2.dengxiaolong.com/docs/command-line#caddy-reload) 更改正在运行的 Caddy 进程的配置
- [caddy reverse-proxy](https://caddy2.dengxiaolong.com/docs/command-line#caddy-reverse-proxy) 一个简单但可用于生产的 HTTP(S) 反向代理
- [caddy run](https://caddy2.dengxiaolong.com/docs/command-line#caddy-run) 在前台启动 Caddy 进程
- [caddy start](https://caddy2.dengxiaolong.com/docs/command-line#caddy-start) 在后台启动 Caddy 进程
- [caddy stop](https://caddy2.dengxiaolong.com/docs/command-line#caddy-stop) 停止正在运行的 Caddy 进程
- [caddy trust](https://caddy2.dengxiaolong.com/docs/command-line#caddy-trust) 将证书安装到本地信任存储中
- [caddy untrust](https://caddy2.dengxiaolong.com/docs/command-line#caddy-untrust) 不信任来自本地信任存储的证书
- [caddy upgrade](https://caddy2.dengxiaolong.com/docs/command-line#caddy-upgrade) 将 Caddy 升级到最新版本
- [caddy add-package](https://caddy2.dengxiaolong.com/docs/command-line#caddy-add-package) 将 Caddy 升级到最新版本，添加了额外的插件
- [caddy remove-package](https://caddy2.dengxiaolong.com/docs/command-line#caddy-remove-package) 将 Caddy 升级到最新版本，删除了一些插件
- [caddy validate](https://caddy2.dengxiaolong.com/docs/command-line#caddy-validate) 测试配置文件是否有效
- [caddy version](https://caddy2.dengxiaolong.com/docs/command-line#caddy-version) 打印版本

### 子命令

就是基本命令更加细节的添加

[command-line — Caddy v2中文文档 (dengxiaolong.com)](https://caddy2.dengxiaolong.com/docs/command-line)



## JSON配置结构

[JSON配置结构 - Caddy V2中文文档 (dengxiaolong.com)](https://caddy2.dengxiaolong.com/docs/json/#admin/remote)