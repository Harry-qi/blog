---
title: egg项目总结
date: 2021-09-02 13:40:16
tags: egg
---
> 最近用egg做了一个完整的后端项目，借此总结下项目中用到的重要知识点
# 1. 跨域问题
之前一直使用`egg-cors`，配置如下，这也是[文档](https://www.npmjs.com/package/egg-cors#configuration)中默认配置
``` javascript
// config.default.js
config.cors = {
    origin: '*',
    // {string|Function} origin: '*',
    // {string|Array} allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH'
};
```
后面有个需求，在另外一个网站里调用本项目的接口，一直报跨域的错，错误如下
![报错](https://i.loli.net/2021/09/02/rCdG924tZJcTnuB.png)
大致意思就是有`withcredentials`的时候不能设置`*`。
看了下`egg-cors`文档没写withcredentials的相关配置，但是我注意到`egg-cors`是基于@koa/cors的，就看到@koa/cors的[文档](https://github.com/koajs/cors#corsoptions)
修改配置如下

``` javascript
// config.default.js
config.cors = {
    origin: 'https://www.xxx.com',
    allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH',
    // Access-Control-Allow-Credentials
    credentials: true,
};
```

这样在xxx网站上调用接口是没有报错的，但是我这个项目还有其他网站调用的，所以我改下`origin`为
``` javascript
// config.default.js
config.cors = {
    origin: 'https://www.xxx.com,'http://localhost:8080','https://www.bbb.cn'',
    allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH',
    // Access-Control-Allow-Credentials
    credentials: true,
};
```
运行了下，直接报错了。查了相关资料`origin`是不能这样设置的。在`egg-cors`中设置多个`origin`需要如下设置
``` javascript
 config.security = {
    csrf: {
      enable: false,
    },
    domainWhiteList: [ 'https://www.xx.com', 'http://localhost:8080', 'https://www.bbb.cn' ],
  };
    // 配置跨域
  config.cors = {
    origin: '*',
    allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH',
    // Access-Control-Allow-Credentials
    credentials: true,
  };
```
至此，跨域问题已经解决。

# 2. 联表查询
> 之前都是获取到数据，再循环数据再次查询数据库，这样做法确实不优雅，于是查询相关资料如何联表查询
业务场景：动态中有多个评论，所以我要根据动态id查询该动态内容以及所有评论
``` javascript
// model/sf_activity.js
'use strict';
module.exports = app => {
    // 定义字段
    //  const Model = app.model.define(...)
    Model.associate = function() {
        // 一个动态有多个评论
        app.model.SfActivity.hasMany(app.model.SfComment, { foreignKey: 'activity_id', targetKey: 'id', as: 'commentList' });
    };
}
```
```javascript
// model/sf_comment.js
'use strict';
module.exports = app => {
    // 定义字段
    //  const Model = app.model.define(...)
    // 评论属于动态
     Model.associate = function() {
        app.model.SfComment.belongsTo(app.model.SfActivity, { foreignKey: 'activity_id', targetKey: 'id', as: 'commentList' });
    };
}
```
`foreignKey`是外键，`targetKey`是外键对应的目标字段 

使用联表查询
```javascript
  get SfActivity() {
    return this.ctx.model.SfActivity;
  }
  get SfComment() {
    return this.ctx.model.SfComment;
  }
 async findList({ page = 1, pageSize = 10 }) {
    const data = await this.SfActivity.findAndCountAll({
      order: [[ 'update_time', 'DESC' ]],
      where: {
        is_delete: 0,
      },
      attributes: [ 'user_name', 'dept_id', 'form_content', 'update_time' ],
      include: {
        model: this.SfComment,
        as: 'commentList',
        attributes: [ 'content', 'create_user_name', 'parent_comment_id' ],
        where: {
          is_delete: 0,
        },
      },
      offset: (page - 1) * pageSize,
      limit: pageSize * 1,
    });
    return data;
}
```
注意：
1. 用`include`字段使用联表查询
2.`as`是别名，需要在`sf_activity.js`和`sf_comment.js`中写，注意一定要统一
3. `attributes`返回接口用这个筛选部分字段，默认不写`attributes`就是查的所有字段
结果如下
![结果](https://i.loli.net/2021/09/02/qGhb7PWxzIv5lme.png)
# 3. 自动创建数据库字段
使用`egg-sequelize-auto`
代码如下
```javascript
// auto.js
'use strict';

const SequelizeAuto = require('egg-sequelize-auto');
const devConfig = require('./config/config.local.js');
const db = {
  database: devConfig.sequelize.database,
  host: devConfig.sequelize.host,
  port: devConfig.sequelize.port,
  username: devConfig.sequelize.username,
  password: devConfig.sequelize.password,
  dialect: devConfig.sequelize.dialect,
};

const auto = new SequelizeAuto(db.database, db.username, db.password, {
  host: db.host,
  dialect: db.dialect,
  directory: './app/model/', // 输出文件夹
  port: db.port,
  additional: {
    timestamps: false,
  },
  tables: [ 'sf_activity', 'sf_comment', 'sf_like', 'sf_read' ], // 指定的表
});

auto.run(function(err) {
  if (err) throw err;
});

```
注意
1. 用`tables`来指定部分需要写入的表(有的时候不一定用得上数据库中所有表)
再在`package.json`中增加命令
```javascript
"scripts":{
    "model": "node auto.js"
}
```
效果如下
![结果](https://i.loli.net/2021/09/02/4Yai6EDQosHzcmf.png)
注意点：
1. `egg`中的eslint默认要在第一行增加`'use strict';`，但是`egg-sequelize-auto`是没有的，需要手动加
2. 生成代码`sequelize`未定义，如下
![问题](https://i.loli.net/2021/09/02/c4gfGKW8zuL1lXp.png)
需要手动修改为app.Sequelize
# 4. 格式化时间
> 默认情况下时间是 2021-09-07T14:15:34.000Z 这种格式，看起来很别扭，一般需要格式化成：2021-09-07 14:15:34
只需要增加`config/local.js`（以及其他环境中），的`dialectOptions`配置
```javascript
exports.sequelize = {
  dialect: 'xx',
  username: 'xx',
  password: 'xxx',
  host: 'xxxxx',
  port: 3306,
  database: 'xxx',
  timezone: '+08:00',
  define: {
    timestamps: false,
    freezeTableName: true,
  },
  dialectOptions: {
    dateStrings: true,
    typeCast: true
  },
};
```
# 5.查看日志
定义日志名称以及位置:
``` javascript
// config/default.js
config.logger = {
  dir: 'logs/egg-for-xx',
};
```
服务器查看日志
进入日志路径 
``` shell
cd /home/xxx/logs/xxx
# 查看倒数100行
tail -fn 100 common-error.log
# 或者
tail -fn 100 egg-for-xx-web.log 
```
