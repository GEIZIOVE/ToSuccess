## NUXT环境初始化

### 2.1 下载压缩包

https://github.com/nuxt-community/starter-template/archive/master.zip

### 2.2解压

将template中的内容复制到 yygh-site

### 2.3修改package.json

name、description、author（必须修改这里，否则项目无法安装）



```vue
module.exports = {

 /*

 ** Headers of the page

 */
 head: {

  title: 'yygh-site',

  meta: [

   { charset: 'utf-8' },

   { name: 'viewport', content: 'width=device-width, initial-scale=1' },

   { hid: 'description', name: 'description', content: '尚医通' }

  ],
 …
}
```



### 2.4修改nuxt.config.js

修改title: '{{ name }}'、content: '{{escape description }}'

这里的设置最后会显示在页面标题栏和meta数据中

module.exports = {

 /*

 ** Headers of the page

 */

 head: {

  title: 'yygh-site',

  meta: [

   { charset: 'utf-8' },

   { name: 'viewport', content: 'width=device-width, initial-scale=1' },

   { hid: 'description', name: 'description', content: '尚医通' }

  ],
 …  

 

### 2.5终端中进入项目目录安装依赖

npm install

![img](file:///C:\Users\Lenovo\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg)

### 2.6 引入element-ui

1、下载element-ui

  npm  install  element-ui  

 

2、在plugins文件夹下创建myPlugin.js文件

![img](file:///C:\Users\Lenovo\AppData\Local\Temp\msohtmlclip1\01\clip_image004.jpg)

 

3、在myPlugin.js文件引入element-ui

import Vue from 'vue'

import ElementUI from 'element-ui' //element-ui的全部组件

import 'element-ui/lib/theme-chalk/index.css'//element-ui的css

Vue.use(ElementUI) //使用elementUI

 

4、在nuxt.config.js文件中使用myPlugin.js

在build下面添加内容：

plugins: [

  { src: '~/plugins/myPlugin.js', ssr: false }

 ]

### 2.7 测试运行

npm run dev

访问项目：http://localhost:3000/

### 2.8 NUXT目录结构

1、资源目录 assets

 用于组织未编译的静态资源如 LESS、SASS 或 JavaScript。

2、组件目录 components

用于组织应用的 Vue.js 组件。Nuxt.js 不会扩展增强该目录下 Vue.js 组件，即这些组件不会像页面组件那样有 asyncData 方法的特性。

3、布局目录 layouts

用于组织应用的布局组件。

4、页面目录 pages

用于组织应用的路由及视图。Nuxt.js 框架读取该目录下所有的 .vue 文件并自动生成对应的路由配置。

5、插件目录 plugins

用于组织那些需要在 根vue.js应用 实例化之前需要运行的 Javascript 插件。

6、nuxt.config.js 文件

nuxt.config.js 文件用于组织Nuxt.js 应用的个性化配置，以便覆盖默认配置。

### 3.1 安装axios

  执行安装命令  

​	npm install axios  

 

### 3.2 封装axios

import axios from 'axios'

import { MessageBox, Message } from 'element-ui'

// 创建axios实例

const service = axios.create({

  baseURL: 'http://localhost',

  timeout: 15000 // 请求超时时间

})

// http request 拦截器

service.interceptors.request.use(

  config => {

  // token 先不处理，后续使用时在完善

  return config

},

 err => {

  return Promise.reject(err)

})

// http response 拦截器

service.interceptors.response.use(

  response => {

​    if (response.data.code !== 200) {

​      Message({

​        message: response.data.message,

​        type: 'error',

​        duration: 5 * 1000

​      })

​      return Promise.reject(response.data)

​    } else {

​      return response.data

​    }

  },

  error => {

​    return Promise.reject(error.response)

})

export default service