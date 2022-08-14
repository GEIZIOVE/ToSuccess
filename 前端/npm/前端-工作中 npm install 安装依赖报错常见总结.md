# 前端-工作中 npm install 安装依赖报错常见总结

# 前端-npm install 安装依赖报错解决

1. 先清除缓存

npm cache clean --force

1. 删除项目中的node_modules文件夹
2. 安装淘宝镜像cnpm，用cnpm来安装依赖

npm install -g cnpm --registry=https://registry.npm.taobao.org

1. 最后再执行

cnpm install

### 报错The package-lock.json file was created with an old version of npm

问题背景：
npm install 报：
The package-lock.json file was created with an old version of npm

问题分析：
你使用npm太新了

解决方法：

```bash
#npm i npm@6 -g
#npm -v
6.14.15
123
```

#### package-lock.json 文件是版本锁定文件

> package-lock.json 是在 `npm install` 时候生成的一份文件，用以记录当前状态下实际安装的各个 npm package 的具体来源和版本号。

package-lock.json的作用：

锁版本，确保项目在后续拉去中，安装依赖包时依赖包的版本始终是统一的

在npm install时会自动生成package-lock.json

package.json与package-lock.json相同时，npm安装包时以package-lock.json为准

package-lock.json与package.json的不同：

package-lock.json记录的是依赖树，记录了依赖模块之间的完整依赖关系。package.json记录的是依赖项，不能锁定依赖的依赖。

总结： package-lock.json 文件的作用锁定安装时的包的版本号，并且需要上传到 git，以保证其他人在 `npm install` 时大家的依赖能保证一致。

### 报错：npm ERR! Maximum call stack size exceeded

问题背景：
npm install 报：npm ERR! Maximum call stack size exceeded

问题分析：

解决办法：

删除node_module文件夹和package-lock.json文件，再重新npm install

sudo rm -rf node_modules package-lock.json
sudo npm install

### npm报错：npm ERR! cb.apply is not a function

win + r 打开运行，输入%appdata%
删除 npm 和 npm-cache 文件夹
执行npm cache clean --force命令

此时应该就可以了。如果还不行，就执行卸载[Node](https://so.csdn.net/so/search?q=Node&spm=1001.2101.3001.7020).js重新安装。

### npm install 报错 check python checking for Python executable python2 in the PATH

问题：
npm install 报错 check python checking for Python executable python2 in the PATH

问题分析：
缺少python2.7支持

解决方法：

```bash
npm install --global windows-build-tools --save
1
```

这个命令会自动安装 python

### npm ERR! node-sass@4.14.1 postinstall: `node scripts/build.js`

出现原因：估计是网络不好下载安装不了，可以使用cnpm（npm淘宝镜像）安装或者手工设置该插件仓库地址

解决方案：

```bash
npm install -g cnpm --registry=https://registry.npmmirror.com
cnpm install
```