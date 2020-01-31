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
* 官方api
  * [Nuxt](https://zh.nuxtjs.org/)
  
## 开始

### ***创建项目***
  ```
  npx create-nuxt-app <项目名>
  ```
  **注意：npx是npm 5.2.0以后新加的命令行，和全局安装 create-nuxt-app 然后再创建项目是一个道理**

### ***我用的版本***
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

### ***别名***  

  |　　　　　别名　　　　　|　　　　　目录　　　　　|
  | :-: | :-: |
  |　　　　　`~`或`@`　　　　　|　　　　　src目录　　　　　|
  |　　　　　`~~`或`@@`　　　　　|　　　　　根目录　　　　　|
 
### ***nuxt.config.js***（这是一个全局配置文件，整合了很多插件的配置文件，例如：webpack，vue）
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
    ]
    ```
    ```
    plugins: [{//引入插件、自己写的一些配置文件
      src: '@/plugins/element-ui',
      mode: 'client',
      ssr: true
      },
      '@/plugins/axios',
      '@/plugins/host'//下面会说简单的注入方式
    ]
    ```
    ```
    build: {
      transpile: [/^element-ui/],
      vendor: ['element-ui', 'axios'],//之前看到有文章说webpack4.？已经自动包含这块打包，这个配置貌似没什么用，具体没有研究，不过我觉得不要不要在这块死磕
    }
    ```
    ```
    router: {//简单明了，路由，nuxt的路由其实就是对vue-router进行封装，配置跟vue-router差不多
      mode:'hash',//路由模式，hash和history
      linkExactActiveClass: 'active',//选中路由添加的class类名
      extendRoutes(routes, resolve) {
        routes.forEach(element => {//这里因为项目有需求，做伪静态，所以我匹配了路由地址在后面加了.html（伪静态自行度娘）
          if (element.path != '/') {
            let path = element.path;
            path = path + '.html';
            element.path = path;
          }
        })
      }
    }
    ```
    ```
    env:{//环境对应地址配置，我用来引入
      baseUrl: process.env.BASE_URL || 'https://xxx.xxx.com'
    }
    ```

### ***插件简单的注入方式***
  * 如果你需要在项目里全局使用某个函数或属性值，就需要将这个函数或属性值注入到vue或nuxt
  * 我比较懒，干脆都注入了，单独注入的方式可以参考官方api，往上看有链接
    ```
    //上面说的host.js配置
    export default ({
      app
    }, inject) => {
      inject('orgHost', process.env.baseUrl);
    }
    ```
    **注意：inject是vue+nuxt同时注入的方法，为什么会有这两种看上去不同的环境的注入？因为nuxt获取数据的方法asyncData它是在vue实例创建之前调用的，所以用Vue.prototype注入的函数或属性值在asyncData里无法调用。so，为什么，就是因为这个**
    
### ***asyncData***
  * 这是一个组件渲染之前调用的异步获取数据的方法
    ```
    async asyncData({ app, params, route }) {//目前我只用了这3个参数，从上下文中结构出来，参数就是字面上的意思
      let [res1, res2] = await Promise.all([
        app.http.post('/api1'),//这里我把axios注入到一个叫http的属性，实质上跟axios.post('/api')是一样的
        app.http.post('/api2', data)
      ])
      let path = route.path;//获取路由地址

      //当然，你也可以写成
      /*
      app.http.post('/api').then(res=>{
        <!-- 请求返回后要执行的代码 -->
      }).catch(err=>{
        <!-- 请求返回后要执行的代码 -->
      })
      */

      return {//讲结果以对象的形式返回，这个时候在页面可以直接绑定，用vue的方法，比如v-bind、v-model、{{resData1}}等等
        resData1,
        resData2
      }
    }
    ```
    **注意：async、await是一对好基友，要同时出现**
  
### ***no-ssr***（有时候某些组件在ssr下显示有问题或者报错，不妨试试no-ssr）
  ```
  <no-ssr placeholder="Loading...">
    <!-- 此组件仅在客户端呈现 -->
  </no-ssr>
  ```
  
### ***自定义头部内容***
  * seo可能会要求不同的页面有它专属的title，keywords，description，head()这个方法是Nuxt自带的自定义头部的方法
    ```
    head() {
      return {
        title: '标题',
        meta: [
          {
            name: 'keywords',
            content: '关键字'
          }
          //anyway，你可以添加更多，像nuxt.config.js里面描述的一样
        ]
      }
    }
    ```

### ***其他***（暂时想不起还有什么东西要说了）
  * 跳链接：用习惯了vue-router可能都会写router-link，nuxt经过封装以后改成了nuxt-link
  * 修改默认模板
    ```
    //在根目录下创建一个app.html，内容如下，这个是默认的，如果想要在模板里添加一些别的元素，在里面写上，生成出来的页面就会默认带有自定义添加的内容啦
    <!DOCTYPE html>
    <html {{ HTML_ATTRS }}>

    <head {{ HEAD_ATTRS }}>
      {{ HEAD }}
    </head>

    <body {{ BODY_ATTRS }}>
      {{ APP }}
    </body>

    </html>
    ```
  
* 打包
  * 项目依赖里能看到一个叫cross-env的东西，它是创建项目时自带安装的，其实...它是个神器
    ```
    //package.json
    "scripts": {
      "dev": "nuxt",
      "build": "nuxt build",
      "start": "nuxt start",
      "generate": "nuxt generate",
      "buildpro": "cross-env BASE_URL=https://xxx.xxx.com nuxt build",//他可以直接把BASE_URL改成某个地址，然后打包，如果项目里用到了BASE_URL，这个是非常方便的一键替换的方法
      "devlocal": "cross-env BASE_URL=http://localhost:8888 nuxt"//开发的时候想改个地址运行也行
    }
    ```
---

## 关于Tornado
  * 它是什么？
    * 是一种开源的 Web 服务器软件
  * 说人话
    * 可以用python语言写一个服务端，配合前端html实现一个网站，类似Flask（自行百度=。=）
  * 官网
    * [Tornado](https://www.tornadoweb.org/en/stable/)
    * [中文版api](https://www.osgeo.cn/tornado/index.html)

## 开始

### ***熟悉的hello world！***
  * 参考
    * [hello world](https://www.tornadoweb.org/en/stable/guide/structure.html)
  * 代码
  ```
  import tornado.ioloop
  import tornado.web

  class MainHandler(tornado.web.RequestHandler):
      def get(self):
          self.write("Hello, world")

  def make_app():
      return tornado.web.Application([
          (r"/", MainHandler),
      ])

  if __name__ == "__main__":
      app = make_app()
      app.listen(8888)
      tornado.ioloop.IOLoop.current().start()
  ```
  * 来点不一样的
  ```
  # 上述Application里注册的，有点类似于前端Vue的Router，当然我们也可以给它一些不同的配置。
  # debug开启，就可以实现热更新，不需要每次修改完都要手动重启。

  settings = {
    "cookie_secret": "\xecE\x15\x13\x94\xb4\x80\xc3\xdd&4\x81s\xe4\x9f\x1d\x160(\xf5\x1b\xe9\xe8\xfc",
    "template_path": "server/templates",
    "static_path": "server/static",
    "debug": True
  }

  def make_app():
      return tornado.web.Application([
          (r"/", MainHandler),
      ],**settings)
  ```

### ***创建一个多线程***
  * 有什么用？
    * 为了一键启动python程序和tornado程序，同时避免跨域问题发生。
  * 直接上代码
  ```
  # 创建一个server，make_app()返回了tornado.web.Application，可以参照上面demo。
  # 多线程存在一个ioloop的问题，此处引入asyncio即可解决。

  import tornado.ioloop
  import tornado.web
  import asyncio

  def start_server():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
  
  def run():
    threading.Thread(target=start_server).start()
  ```

### ***分享一个长轮询***
  * 目标
    * 做一个聊天室
  * 需求
    * a和b聊天，当a发送内容后，b的界面不需要刷新，就能看到最新消息。
  * 旧轮询方案
    * 实现：
      * 前端设置一个定时器，setInterval，假定3秒执行一次。
    * 弊端
      * 如果a说话比较频繁，可能3秒后会弹出非常多条信息。
      * 如果a和b之间只是打开了聊天窗（其实只有一个人打开也是同样道理），并没有做交流，定时器依然会不停的做轮询，做无用功。
  * 新轮询方案
    * 实现
      * 创建一个类（***MessageBuffer***），处理消息（添加消息、获取消息）。
      * 创建一个类（***ChatHandler***），处理发送按钮事件（类似前端接口），实际上只是拿到新消息然后交给 MessageBuffer 进行消息添加。
      * 创建一个类（***UpdateMsgHandler***），负责向 MessageBuffer 获取消息
    * 服务器端
    ```
    class MessageBuffer(object):

      def __init__(self):
          # 类似线程锁
          self.cond = tornado.locks.Condition()
          # 存放消息
          self.cache = []

      def get_msg_since(self, cursor):
          results = []
          # 把数组翻转，从最后一个取，当取到id与cursor相同时，跳出循环。
          # id 是区分每一条消息的唯一标识
          # cursor 是已经推送的消息里最后一条的id
          for msg in reversed(self.cache):
              if msg["id"] == cursor:
                  break
              results.append(msg)
          results.reverse()
          return results

      def add_msg(self, message):
            self.cache.append(msg)
            # 叫醒被锁线程
            self.cond.notify_all()

    # 创建一个实例
    msgBuffer = MessageBuffer()

    class ChatHandler(tornado.web.RequestHandler):
      def post(self):
          # 获取客户端传入的消息
          query = self.get_argument('query', default='')
          if query != '':
              # id 是生成的一个随机数
              message = {'id': uuid.uuid4(), 'text': query}
              msgBuffer.add_msg(message)

    class UpdateMsgHandler(tornado.web.RequestHandler):
      async def post(self):
          # cursor 是已推送消息里最后一条的id
          cursor = self.get_argument('cursor', default=None)
          messages = msgBuffer.get_msg_since(cursor)
          while not messages:
              # 创建一个阻塞（个人理解）
              self.wait_future = msgBuffer.cond.wait()
              try:
                  await self.wait_future
              except asyncio.CancelledError:
                  return
              messages = msgBuffer.get_msg_since(cursor)
          res = {'code': 0, 'message': 'ok', 'history': messages}
          self.write(json.dumps(res))

    # 别忘记把两个继承自tornado中RequestHandler的类注册到Application中
    def make_app():
      return tornado.web.Application([
          (r'/chat', ChatHandler),
          (r'/updatemsg', UpdateMsgHandler)
      ])
    ```
      * 客户端
    ```
    var updater = {
        cursor:null,

        poll: function () {
            $.ajax({
                type: 'post',
                url: '/updatemsg',
                data:{
                    cursor:updater.cursor
                },
                success: function (res) {
                    var data = JSON.parse(res)
                    if (data.code == 0) {
                        //在这添加需要处理的逻辑
                        //将cursor设置为已推送最后一条消息的id
                        updater.cursor = data.history[data.history.length-1].id
                        //然后再制造下一个等待线程
                        window.setTimeout(updater.poll, 0);
                    }
                },
                error: function (err) {
                    console.log(err)
                }
            })
        }
    }
    ```

    * 步骤
      * 页面打开时请求/updatemsg，cursor的初始值可以是null（因为一开始消息列表是空的，链接了数据库除外）
      * 这时候 UpdateMsgHandler 中就有一个等待的线程
      * 输入一条消息提交到/chat，ChatHandler 里会触发 add_msg 方法，add_msg方法里有一个唤醒机制，叫醒在等待的线程
      * 这时候 UpdateMsgHandler 里的线程就会调用 get_msg_since 方法得到最新的消息列表返回到客户端，同时跳出while循环
      * 客户端拿到接口返回值以后进行页面渲染，***同时别忘了再次请求/updatemsg（制造下一个等待线程）***