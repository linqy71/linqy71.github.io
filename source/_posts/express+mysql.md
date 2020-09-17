---
title: nodejs + express + mysql搭建restful风格api
date: 2019-01-18 2:03:23
tags: express nodejs mysql webapi
categories: nodejs
copyright: true
---

#### 开发前准备

1. 安装node和express
2. 命令行输入express + 项目名初始化项目
`> express server`
3. 进入server文件夹安装依赖
`> npm install`
4. 由于要使用mysql，因此，在package.json中的`"dependencies"`下添加`"mysql": "latest"`，重新执行`npm install`

#### 服务器上安装mysql
1. 安装mysql
 CentOS的yum源中没有mysql。没法直接 yum -y install


**正确方法**：参考[CentOS7 64位安装mysql教程，亲测完美](https://blog.csdn.net/a774630093/article/details/79270080)，包括一些初始配置也在里面。（里面还有开放root供远程连接的方法，保存备用。）



#### 数据库相关

1. 项目下新建database文件夹，新建DBConfig.js，写入：
    ```
    module.exports = {
      mysql : {
        host : '127.0.0.1',
        user : 'root',
        password : yourPSW,
        database : yourDB,
        port : 3306
      }
    }
    ```

2. 创建数据库和表
     创建数据库：启动mysql后输入 `create database kuailing`
     创建表：采用导入.sql文件的形式。
     ![](https://i.loli.net/2019/01/16/5c3f1b16529b7.png)

3. 封装一些数据库操作语句。（我将封装块放在了userDAO中）
    ```
    var userSqlMap = {
      add: 'insert into user(studentid, name, realname, address, password, phone)' +
           ' values(?, ?, ?, ?, ?, ?)',
      deleteById: 'delete from user where studentid = ?',
      update: 'update user set name=?, realname=?, address=?, password=?' +
           ', phone=? where studentid=?',
      list: 'select * from user',
      getById: 'select * from user where studentid=?'
    };
    ```

4. 封装返回结果对象。（非必要，因为想要实现api接口无需体现状态）
     在model文件夹下新建result.js，返回操作结果success以及需要的数据。
    ```
    exports.createResult = function(success, data) {
      var result = {};
      result.success = success;
      result.data = data;
      return result;
    };
    ```


5. 封装和数据库的交互模块。
 新建DAO文件夹，新建userDAO.js，使用连接池防止频繁获取、释放数据库连接。写入的部分代码如下: 
    ```
    //建立连接池
    var pool = mysql.createPool(DBConfig.mysql);
    module.exports = {
      getById: function (studentid, callback) {
        pool.query(userSqlMap.getById, studentid, function (err, result) {
          if (err) throw err;
          console.log(result[0]);
          callback(result[0]);
        });
      },
      //添加新用户
      add: function (user, callback) {
        user = JSON.parse(user);
        pool.query(userSqlMap.add, [user.studentid, user.name, user.realname, user.address,
         user.password, user.phone], function (err, result) {
           if (err) throw err;
           console.log('result :' + result);
           callback(result.affectedRows > 0);
         });
      }
    };
    ```


#### 业务逻辑

1. 用户操作路由及实现业务逻辑routes/users.js:

  - GET

    ```
    //解析 /users?studentid=xxxx的url
    router.get('', function(req, res) {
      console.log(req.query);
      userDAO.getById(req.query.studentid, function (user) {
        console.log(user);
        res.json(user);
      });
    });
    ```

  - POST

    ```
    //提交用户信息用于添加新用户
    router.post('/', function(req, res) {
      var user = JSON.stringify(req.body);
      userDAO.add(user, function(success) {
        var result = {};
        result.success = success;
        res.json(result);
      });
    });
    ```

  - /users/xxxx的url,获取查询参数的方式是req.params
  
    ```
    //解析 /users/xxxx的url，其中xxxx为学号,
    //可以采用以下方式，获取查询参数的方式是req.params。
    router.get('/:studentid', function (req, res) {
      console.log('patch users called');
      userDAO.getById(req.params.studentid, function (user) {
        console.log(user);
        res.json(user);
      });
    });
    ```


#### 遇到的问题
- 启动mysql(`mysql -u root -p`)后使用数据库mysql(`use mysql`)输入`show variables like "%char%";`想要确定是否为utf8，出现报错：ERROR 1146 (42S02): Table 'performance_schema.session_variables' doesn't exis。

    *解决方法*：
    
    输入：`mysql_upgrade -u root -p --force`并重启服务。
    效果：
    ![](https://i.loli.net/2019/01/16/5c3f0fa9bf4fd.png)


- 由于以前用的都是mysql workbench，不熟悉命令行操作mysql，因此，命令行操作参考：
[https://www.cnblogs.com/kpengfang/p/5201285.html](https://www.cnblogs.com/kpengfang/p/5201285.html)


- 关于采用post实现新增用户一直返回失败的问题。

    *解决方法*：
    
     获取用户信息是通过`var user = JSON.stringify(req.body);`，然后将这个user传递到userDAO的add方法中，`user.name`却变成undefine，在add方法中添加`user = JSON.parse(user);`将参数转化为JSON对象就解决啦。


### 总结

以上，就完成了基本GET和POST方式的api。其余更多需要的处理在于对数据库的设计和操作，依据项目要求而进行调整。

后来查看文档发现还是有很多地方不符合规范，需要进一步的改进，参考[restful api设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html "restful api设计指南")

### 参考

[https://www.cnblogs.com/thinkam/p/8299452.html](https://www.cnblogs.com/thinkam/p/8299452.html)