---
title: egg.js学习
date: 2023-5-22 13:51
categories: javaScript
---
# 环境搭建

使用以下命令即可安装egg环境

```bash
yarn create egg --type=simple --registry=china
```

下载之后安装依赖，然后命令行输入`npm run dev`打开页面即可看到页面上的hi egg！

# 连接数据库

先安装数据库

```bash
cnpm i egg-mysql -S
```

安装过后要在egg.js自带框架的config文件夹中添加配置，如果在app文件夹下再创config文件夹，外部文件无法使用，可能会出现错误

plugin.js

```js
module.exports = {
  mysql: {
      enable: true,
      package: 'egg-mysql',
  }
};
```

config.default.js

```js
config.mysql = {
    app: true, //是否挂载到app下
    agent: false, // 是否挂载到代理下
    client: { // 连接数据库信息
      host: '127.0.0.1',
      port: '3306',
      user: '你的账号(通常root)',
      password: '你的密码',
      database: '你的数据库名字',
    }
  }
```

如此一来就可以连接到数据库，但在使用router引入controller的使用，一定要注意以下两点：

- 每个controller是否module.exports抛出
- 每个引入的controller名字是否对应，否则会报错

这个真的算是简单的大坑了，浪费了很多时间

# 路由的封装

随着项目庞大路由可能会越来越多，那么分配路由可能是更好的选择，如以下代码

app/router.js

```js
'use strict';

/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
  require('./router/girl')(app)
  require('./router/index')(app)
};

```

app/router/index.js

```js
'use script'

module.exports = app => {
    const { router, controller } = app;
    router.get('/', controller.home.index);
    router.get('/getData', controller.home.getData)
}
```

app/router/girl.js

```js
'use script'

module.exports = app => {
    const { router, controller } = app;
    router.post('/addgirl', controller.girlsManage.addGirl);
    router.post('/delgirl', controller.girlsManage.delGirl);
    router.post('/updategirl', controller.girlsManage.updateGirl);
    router.get('/getgirls', controller.girlsManage.getGirls);
}
```

如果更复杂的项目，以上同理即可

# controller和service的简单使用例子

controller/girlsManage.js

```js
async getGirls() {
        const { ctx } = this;
        const res = await ctx.service.grils.getGirls();
        ctx.body = {
            code: 200,
            msg: '查询女孩:',
            data: res
        }
    }
```

service/girls.js

```js
async getGirls() {
        try {
            const app = this.app;
            const res = await app.mysql.select('grils');
            return res;
        } catch (error) {
            console.log(error);
            return null
        }
    }
```

可以看出controller只处理前端调取后会显示的数据，并且向service获取数据，至于如何处理数据，有时候还会传参，参数比较处理数据等，都是在service来完成的，要养成这种习惯，把controller当作成一个桥梁即可

# 跨域的配置

使用cors来进行跨域配置，浏览器会在请求头中发送origin字段指明请求发起的地址，服务端返回Access-control-allow-origin，如果一致则可以进行跨域访问

先安装cors插件

```bash
cnpm install egg-cors -S
```

安装之后需要在config文件夹中的两个文件进行配置

plugin.js

```js
cors: {
    enable: true,
    package: 'egg-cors'
  }
```

config.default.ts

```js
// 配置跨域
  config.cors = {
    // 允许请求的来源，为*表示允许所有IP请求
    origin: '*',
    // 允许请求的方式
    allowMethods: "GET, HEAD, PUT, POST, DELETE, PATCH",
    credentials: true
  }
  // 配置对象安全性上定义白名单域属性
  config.security = {
    domainWhiteList: ["*"]
  }
```

如此配置完过后就不会再出现跨域问题了

# 登录注册的编写

## 注册

编写注册接口需要考虑以下几种情况：

- 信息不可以为空

  controller/user.js

  ```js
  const {username, password, name} = ctx.request.body;
        if(!username || !password || !name) {
          ctx.body = {
            code: 500,
            msg: '信息不可以为空',
            data: null
          }
          return
        }
  ```

- 检测用户名是否已存在

  service/user.js

  ```js
  async getUserByName(username) {
      try {
        const result = await this.app.mysql.get('user', { username })
        return result;
      } catch(error) {
        console.log(error);
        return null;
      }
    }
  ```

  controller/user.js

  ```js
   const userInfo = await ctx.service.user.getUserByName(username);
        if(userInfo && userInfo.id) {
          ctx.body = {
            code: 500,
            msg: '用户名已被注册, 请重新输入',
            data: null
          }
          return
        }
  ```

  在service下的user.js文件中书写根据用户名查找数据操作，如果存在数据，说明此用户名已被注册，应当重新注册

- 将前端传递过来的数据放到数据库中

  service/user.js

  ```js
  async register(registerInfo) {
      try{
        const result = await this.app.mysql.insert('user', registerInfo);
        return result
      } catch(error) {
        console.log(error);
        return null
      }
    }
  ```

  controller/user.js

  ```js
   let result = await ctx.service.user.register({
          name,
          password,
          username,
          job: '用户'
        });
        if(result) {
          ctx.body = {
            code: 200,
            msg: '注册成功!',
            data: null
          }
        } else {
          ctx.body = {
            code: 500,
            msg: '注册失败',
            data: null
          }
        }
  ```

  将数据填入收到数据库后，如果没有异常，则可以向前端返回一个200状态码

## 登录

编写登录接口需要将传过来的username当作查询对象向数据库请求是否存在此条数据

service/user.js

```
async login(username) {
    const { ctx, app } = this;
    const userInfo = await app.mysql.get('user', { username });
    return userInfo
  }
```

- 账号不存在

  ```js
  const { username, password } = ctx.request.body;
          // 根据用户名在数据库查找相对应的操作
          const userInfo = await ctx.service.user.login(username);
          // 没找到说明没有该用户
          if(!userInfo || !userInfo.id) {
            ctx.body = {
              code: 500,
              msg: '账号不存在',
              data: null
            }
            return
          }
  ```

- 账号存在但用户名和密码不匹配

  ```js
  if (userInfo && password !== userInfo.password) {
            ctx.body = {
              code: 500,
              msg: '账号或密码错误',
              data: 'null'
            }
            return
          }
  ```

- 账号存在并且用户名和密码匹配。则返回登陆成功状态码并且携带token字段

  ```js
  const token = app.jwt.sign({
              id: userInfo.id,
              username: userInfo.username,
              exp: Math.floor(Date.now()/1000) + (24 * 60 * 60) // token有效期为24小时
            }, app.config.jwt.secret);
            // 返回token
            ctx.body = {
              code: 200,
              msg: '登陆成功!',
              data: { token }
            }
  ```

登录注册的部分代码实现，现在还没有结合权限，所以并不是特别的完善

# middleware实现token鉴权

在中间件书写基本逻辑和异常捕获，然后在router中需要的路由添加此中间件的使用即可。我踩了个大坑，就是如果无论怎么样也捕获不到异常，就检查一下middleware文件夹是否在app文件夹下，这个耽误了很多时间，搜了很多资料都没用。如果以后在项目中遇到废了很长时间还没解决的bug时，大概率就是非常简单的问题。

middleware/jwt.js

```ts
module.exports = (config) => {
    return async function jwt(ctx, next) {
            const token = ctx.request.header.authorization;
            let decode;
            if (token != 'null' && token) {
                try {
                    decode = ctx.app.jwt.verify(token, config.secret); // 验证token
                    await next();
                } catch (error) {
                    ctx.body = {
                        msg: 'token已过期，请重新登录',
                        code: 401,
                    }
                    return;
                }
            } else {
                ctx.body = {
                    code: 500,
                    msg: 'token不存在',
                };
                return;
            }
    }
}
```

router/home.js

```tsx
const { router, controller, middleware } = app;
const jwt = middleware.jwt(app.config.jwt)
router.post('/getData', jwt, controller.home.getData)
```

# egg外部引入文件

egg不能使用import引入外部文件，只能使用require来引入，使用module.exports抛出，即可实现模块化开发
