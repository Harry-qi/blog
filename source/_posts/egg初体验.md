---
title: egg初体验
date: 2021-08-03 16:59:27
tags:
---
> 最近公司准备用node写全栈项目，于是花了半天时间熟悉了`egg`的文档，整体下来体验良好，不得不说`egg`确实不错。

## 快速搭建
``` shell
$ mkdir egg-example && cd egg-example
$ npm init egg --type=simple
$ npm i
```
可以在官网熟悉下[目录结构](https://eggjs.org/zh-cn/basics/structure.html)，知道每个目录的作用。
我自己的理解：
1. `app/controller/**`是来接受入参，调用`service`的方法，得到返回`service`的结果，最后返回结果，即接口最终数据。
2. `app/service/**`是来根据参数查询数据库，对数据进行处理转换等业务逻辑，然后将处理好的数据返回。
3. `app/model/**`是用来定义数据表的每个字段
4. `app/router.js`用来定义接口路径，接口方法(`get`、`post`、`delete`、 `put`等)
5. `config/config.{env}.js`用来设置不同环境的配置，`config.default.js` 为默认的配置文件，所有环境都会加载这个配置文件。`config.prod.js`如果有`config.default.js`同名配置，则以`config.prod.js`为准
6. `config/plugin.js`插件的注册需要在这里写
7. `app/middleware/**`这里写一些中间件，例如，我们需要对返回接口统一数据格式，就可以写个中间件。

## 前置工作
### 1. 安装依赖
``` shell
$ npm i egg-cors egg-mysql egg-sequelize egg-validate mysql2 -S
```
### 2. `mysql`和`sequelize`配置
``` javascript
// config/local.js
'use strict';
exports.mysql = {
  // 单数据库信息配置
  client: {
    // host
    host: 'xxx',
    // 端口号
    port: '3306',
    // 用户名
    user: 'xxx',
    // 密码
    password: 'xxx',
    // 数据库名
    database: 'xxx',
  },
  // 是否加载到 app 上，默认开启
  app: true,
  // 是否加载到 agent 上，默认关闭
  agent: false,
};
exports.sequelize = {
  dialect: 'mysql',
  username: 'xxx', // 用户名
  password: 'xxx', // 用户密码
  host: 'xxx', // 服务器地址
  port: 3306, //端口
  database: 'xxx', //数据库名
  define: {
    timestamps: false,
    freezeTableName: true,
  },
};
```

### 3. 跨域配置、中间件的注册
``` javascript
// 在config.default.js中加入
module.exports = appInfo => {
  // add your middleware config here
  config.middleware = [
    'response',
    'errorHandler',
  ];

  // 配置跨域
  config.cors = {
    origin: '*',
    // {string|Function} origin: '*',
    // {string|Array} allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH'
  };
}
```
``` javascript
// plugin.js
'use strict';

/** @type Egg.EggPlugin */
module.exports = {
  // had enabled by egg
  // static: {
  //   enable: true,
  // }
  sequelize: {
    enable: true,
    package: 'egg-sequelize',
  },
  mysql: {
    enable: true,
    package: 'egg-mysql',
  },
  validate: {
    enable: true,
    package: 'egg-validate',
  },
  cors: {
    enable: true,
    package: 'egg-cors',
  },
};

```
## 开始写个接口
### 1. 路由表中定义接口
``` javascript
// router.js
'use strict';

/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
  const { router, controller } = app;
  const auth = app.middleware.auth();

  // 添加统一的接口前缀
  router.prefix('/api');

  router.get('/', controller.home.index);
  router.get('/user', auth, controller.user.index);
};

```
### 2. 创建`model`，定义数据库
``` javascript
// model/user.js
'use strict';
module.exports = app => {
  const { STRING, INTEGER, TIME, TINYINT } = app.Sequelize;
  const User = app.model.define('user', {
    id: { type: INTEGER, primaryKey: true, autoIncrement: true },
    account: STRING(64),
    password: STRING(64),
    phone: STRING(64),
    create_time: TIME,
    update_time: TIME,
    modify_user_id: INTEGER,
    create_user_id: INTEGER,
    is_delete: TINYINT,
  });
  return User;
};

```

### 3. `service`层写逻辑
``` javascript
// service/user.js
'use strict';
const Service = require('egg').Service;
function toInt(str) {
  if (typeof str === 'number') return str;
  if (!str) return str;
  return parseInt(str, 10) || 0;
}
class UserService extends Service {
  async find(uid) {
    // 假如 我们拿到用户 id 从数据库获取用户详细信息
    const user = await this.ctx.model.User.findByPk(toInt(uid));
    return { user };
  }
}

module.exports = UserService;

```

### 4. `controller`层获取入参，调用`service`的方法并返回数据
``` javascript
// controller/user.js
'use strict';
const Controller = require('egg').Controller;

class UserController extends Controller {
  async index() {
    const ctx = this.ctx;
    // 参数校验
    this.ctx.validate({
      id: { type: 'string',required: true },
    }, ctx.query);

    const userId = ctx.query.id; 
    const user = await ctx.service.user.find(userId);
    ctx.body = user;
    ctx.res.success();
  }
}

module.exports = UserController;

```
注：1. `get`请求使用`this.ctx.query`获取参数；`post`使用`this.ctx.request.body`来获取参数，更多参考[文档](https://eggjs.org/zh-cn/basics/controller.html#%E8%8E%B7%E5%8F%96-http-%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0)
2. 参数校验使用来`egg-validate`，在`plugin.js`中注册了，就可以通过`this.ctx.validate`使用(插件会挂载到ctx上)，如果不满足校验规则，则会给出提示，如下
![错误提示](https://i.loli.net/2021/08/03/MC3SwBPNHuRxzFn.png)
3. `findByPk`是`Sequelize`的方法，更多的api可以查看[文档](https://www.sequelize.com.cn/)
4. 如果在`model`层中未定义字段，使用create或者find等其他方法的时候就不会插入或者查到该字段。**所以如果中途改数据库(例如增加字段，修改字段等操作)，一定要在`model`层及时修改**

至此，一个简单的查询接口就完成了，效果如下图
![效果图](https://i.loli.net/2021/08/03/DBeo75jcgnzFh1E.png)

## 总结
总体下来明白了`model`、`service`、`controller`每层的作用，写的时候就清晰很多，更多的就是熟悉`Sequelize`的`api`来完成业务。
