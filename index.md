# 来啦老弟～

### 这是我的一个学习分享页
### 记录一些踩过的坑，填坑很难，保佑日后bug越来越少🙏
---

## 关于 Nuxt

* 它是什么？
  * 基于Vue的一个开箱即用的ssr框架
* ssr是什么？
  * 服务器渲染
* 什么是服务器渲染？
  * 跟传统页面相似，右键查看网页源代码能清楚看见该页面的源代码
* 为什么要用Nuxt做服务器渲染？
  * 因为既想用Vue开发项目，又想满足seo需求
* 关于其他
  * 其实Vue官方有关于ssr的解决方案，但是会比较复杂，因此我选了Nuxt。在React上，也有一个叫Next的ssr解决方案，如果有用到会记录一下。
   
## 开始

* 创建项目
  ```
  npx create-nuxt-app <项目名>
  ```
  **注意：npx是npm 5.2.0以后新加的命令行，和全局安装 create-nuxt-app 然后再创建项目是一个道理**

* 我用的版本
  ```
    "dependencies": {
      "@nuxtjs/axios": "^5.3.6",
      "@nuxtjs/proxy": "^1.3.3",//解决axios兼容问题
      "cross-env": "^5.2.0",
      "element-ui": "^2.8.2",//饿了么组件
      "normalize.css": "^8.0.1",//css reset
      "nuxt": "^2.6.3"
    },
    "devDependencies": {
      "nodemon": "^1.18.9",
      "eslint-config-prettier": "^4.1.0",
      "eslint-plugin-prettier": "^3.0.1",
      "prettier": "^1.16.4"
    }
  ```

* 别名
  |别名|目录|
  |-|-|
  |`~`或`@`|src目录|
  |`~~`或`@@`|根目录|
 
* nuxt.config.js（这是一个全局配置文件，整合了很多插件的配置文件，例如：webpack，vue）
    ```
    mode: 'spa'//nuxt模式，spa：单页面，universal：默认
    ```
    **注意：如果是做ssr的话必须要把模式设置为universal或不写这一行，不要想着又做单页面又可以实现ssr**
    ```
    head: {//网站公共头部内容配置，会一直存在在页面源代码里，除非把他在页面里单独设置（后面会说）
    title: '网站title',
    meta: [{
        charset: 'utf-8'
      },
      {
        name: 'viewport',
        content: 'width=device-width, initial-scale=1'
      },
      {
        hid: 'description',
        name: 'description',
        content: pkg.description
      },
      {
        'http-equiv': 'X-UA-Compatible',
        content: 'IE=edge,chrome=1'
      },
      {
         name: 'keywords',
         content: '网站关键字'
      }],
      link: [{
        rel: 'icon',
        type: 'image/x-icon',
        href: '/logo.ico'
      }],
    }
    ```
    ```
    css: [//公共样式，～是别名，往上看
      'normalize.css',
      'element-ui/lib/theme-chalk/index.css',
      '~/assets/iconfonts/iconfont.css',
      '~/assets/css/common.css'
    ],
    ```
