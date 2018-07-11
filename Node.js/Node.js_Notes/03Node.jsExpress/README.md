---

---

# Node.js - Express 

## 一、模块标识

### 1.文件操作

```javascript
var fs = require('fs')

/*咱们所使用的所有文件操作的【 API 都是异步的】
就像你的 ajax 请求一样
文件操作中的相对路径可以省略 【./】
fs.readFile('data/a.txt', function (err, data) {
  if (err) {
    return console.log('读取失败')
  }
  console.log(data.toString())
})

在模块加载中，相对路径中的 ./ 不能省略
Error: Cannot find module 'data/foo.js'
require('data/foo.js')  //这样是加载不到的

//这个是模块的标识  路径形式的模块 可以第三方 
require('./data/foo.js')('hello')

require('./data/foo')('hello')//加载本地的模块 
在文件操作的相对路径中
   ./data/a.txt 相对于当前目录
   data/a.txt   相对于当前目录
   /data/a.txt  绝对路径，当前文件模块所处磁盘根目录
   c:/xx/xx...  绝对路径
fs.readFile('./data/a.txt', function (err, data) {
  if (err) {
    console.log(err)
    return console.log('读取失败')
  }
  console.log(data.toString())
})

这里如果忽略了 . 则也是磁盘根目录*/
require('/data/foo.js')
```

## 二、Express 初探

### 1.搭建express环境

```shell
npm i express --save
```

### 2.使用express

```javascript
var express = require('express')

// 1. 创建 app 有点类似微信的app 和 page 对象
var app = express()

/*当以 /public/ 开头的时候，去 ./public/ 目录中找找对应的资源
这种方式更容易辨识，推荐这种方式
app.use('/public/', express.static('./public/'))

必须是 /a/puiblic目录中的资源具体路径
app.use('/abc/d/', express.static('./public/'))

当省略第一个参数的时候，则可以通过 省略 /public 的方式来访问
这种方式的好处就是可以省略 /public/*/
app.use(express.static('./public/'))


app.get('/', function (req, res) {
  res.send('hello world')
})

app.listen(3000, function () {
  console.log('express app is running ...')
})

```

### 3.Router路由

```javascript
/*路由其实就是一张表
  这个表里面有具体的映射关系
  xxx 大厦
     xxx 公司
  看门的老大爷
     xxx 公司 15 楼 3 号
     百度公司 4 楼 5 号
     。。。
  经过老大爷，你要去哪个公司

  app
    .get('/login', 函数)
    .get('/dsadsa', 函数)
    .post('/d/sadsa', 函数)
    .get('dsadsa', 函数)*/
```

```javascript
/**
 * router.js 路由模块
 * 职责：
 *   处理路由
 *   根据不同的请求方法+请求路径设置具体的请求处理函数
 * 模块职责要单一，不要乱写
 * 我们划分模块的目的就是为了增强项目代码的可维护性
 * 提升开发效率
 */

var fs = require('fs')
var Student = require('./student')

// Express 提供了一种更好的方式
// 专门用来包装路由的
var express = require('express')

// 1. 创建一个路由容器
var router = express.Router()

// 2. 把路由都挂载到 router 路由容器中

/*
 * 渲染学生列表页面
 */
router.get('/students', function (req, res) {
  Student.find(function (err, students) {
    if (err) {
      return res.status(500).send('Server error.')
    }
    res.render('index.html', {
      fruits: [
        '苹果',
        '香蕉',
        '橘子'
      ],
      students: students
    })
  })
})

/*
 * 渲染添加学生页面
 */
router.get('/students/new', function (req, res) {
  res.render('new.html')
})

/*
 * 处理添加学生
 */
router.post('/students/new', function (req, res) {
  // 1. 获取表单数据
  // 2. 处理
  //    将数据保存到 db.json 文件中用以持久化
  // 3. 发送响应
  Student.save(req.body, function (err) {
    if (err) {
      return res.status(500).send('Server error.')
    }
    res.redirect('/students')
  })
})

/*
 * 渲染编辑学生页面
 */
router.get('/students/edit', function (req, res) {
  // 1. 在客户端的列表页中处理链接问题（需要有 id 参数）
  // 2. 获取要编辑的学生 id
  // 
  // 3. 渲染编辑页面
  //    根据 id 把学生信息查出来
  //    使用模板引擎渲染页面

  Student.findById(parseInt(req.query.id), function (err, student) {
    if (err) {
      return res.status(500).send('Server error.')
    }
    res.render('edit.html', {
      student: student
    })
  })
})

/*
 * 处理编辑学生
 */
router.post('/students/edit', function (req, res) {
  // 1. 获取表单数据
  //    req.body
  // 2. 更新
  //    Student.updateById()
  // 3. 发送响应
  Student.updateById(req.body, function (err) {
    if (err) {
      return res.status(500).send('Server error.')
    }
    res.redirect('/students')
  })
})

/*
 * 处理删除学生
 */
router.get('/students/delete', function (req, res) {
  // 1. 获取要删除的 id
  // 2. 根据 id 执行删除操作
  // 3. 根据操作结果发送响应数据

  Student.deleteById(req.query.id, function (err) {
    if (err) {
      return res.status(500).send('Server error.')
    }
    res.redirect('/students')
  })
})

// 3. 把 router 导出
module.exports = router

/*这样也不方便
module.exports = function (app) {
  app.get('/students', function (req, res) {
    // readFile 的第二个参数是可选的，传入 utf8 就是告诉它把读取到的文件直接按照 utf8 编码转成我们能认识的字符
    // 除了这样来转换之外，也可以通过 data.toString() 的方式
    fs.readFile('./db.json', 'utf8', function (err, data) {
      if (err) {
        return res.status(500).send('Server error.')
      }

      // 从文件中读取到的数据一定是字符串
      // 所以这里一定要手动转成对象
      var students = JSON.parse(data).students

      res.render('index.html', {
        fruits: [
          '苹果',
          '香蕉',
          '橘子'
        ],
        students: students
      })
    })
  })

  app.get('/students/new', function (req, res) {

  })

  app.get('/students/new', function (req, res) {

  })

  app.get('/students/new', function (req, res) {

  })

  app.get('/students/new', function (req, res) {

  })

  app.get('/students/new', function (req, res) {

  })
}*/

```



### 4.第三方插件工具 nodemon

​	nodemon 可以解决频繁的修改代码导致的重启问题，nodemon是一个基于Node.js开发的第三方命令行工具

​	安装如下：

```shell
# --global 安装的为全局的地址  全局都可以起效
npm install --global nodemon

#是否安装成功  
nodemon --version
```

​	使用方式：

> nodemon 使用教程

- node app.js
- nodemon app.js 启动服务会监视文件的变化 自动重启服务器

### 5.静态服务

```javascript
//开放资源 '/'开头的话  去 public 路径找数据
app.use('/',express.static('./public/')) //指定对应的静态的数据资源
/*当以 /public/ 开头的时候，去 ./public/ 目录中找找对应的资源
这种方式更容易辨识，推荐这种方式
app.use('/public/', express.static('./public/'))

必须是 /a/puiblic目录中的资源具体路径
app.use('/abc/d/', express.static('./public/'))

当省略第一个参数的时候，则可以通过 省略 /public 的方式来访问
这种方式的好处就是可以省略 /public/ [数据直接取出]*/ 
app.use(express.static('./public/'))
```

### 6.Express使用

```javascript
/**
 * app.js 入门模块
 * 职责：
 *   创建服务
 *   做一些服务相关配置
 *     模板引擎
 *     body-parser 解析表单 post 请求体
 *     提供静态资源服务
 *   挂载路由
 *   监听端口启动服务
 */

var express = require('express')
var router = require('./router')
var bodyParser = require('body-parser')

var app = express()

app.use('/node_modules/', express.static('./node_modules/'))
app.use('/public/', express.static('./public/'))


//这里是必须安装的
app.engine('html', require('express-art-template'))

// 配置模板引擎和 body-parser 一定要在 app.use(router) 挂载路由之前
// parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }))
// parse application/json
app.use(bodyParser.json())

// 把路由容器挂载到 app 服务中
app.use(router)

app.listen(3000, function () {
  console.log('running 3000...')
})

//暴露出去
module.exports = app
```

## 三、Promise 容器概念

> promise容器

- promise 容器容器中放了一个异步的任务
- 状态只能变成一种
- promise本身并不是异步的 但是内部旺旺都是封装的一个异步的任务

![promise容器概念](C:\Users\Jason\Documents\Front-end-Web\Node.js\Node.js_Notes\03Node.jsExpress\media\promise容器概念.png)

> promise的链式调用

![promise容器概念](C:\Users\Jason\Documents\Front-end-Web\Node.js\Node.js_Notes\03Node.jsExpress\media\promise容器概念.png)

> 回调函数获取异步的操作结果

![回调函数](C:\Users\Jason\Documents\Front-end-Web\Node.js\Node.js_Notes\03Node.jsExpress\media\回调函数.png)

> promise api

![promiseAPI代码图示](C:\Users\Jason\Documents\Front-end-Web\Node.js\Node.js_Notes\03Node.jsExpress\media\promiseAPI代码图示.png)

