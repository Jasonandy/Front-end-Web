# 01Node.js

## 一、Node.js简介

- Node.js 是什么
  + JavaScript 运行时
  + 既不是语言，也不是框架，它是一个平台
- Node.js 中的 JavaScript
  + 没有 BOM、DOM
  + EcmaScript 基本的 JavaScript 语言部分
  + 在 Node 中为 JavaScript 提供了一些服务器级别的 API
    * 文件操作的能力
    * http 服务的能力

## 二、知识结构

- Node 中的 JavaScript
  + EcmaScript
    * 变量
    * 方法
    * 数据类型
    * 内置对象
    * Array
    * Object
    * Date
    * Math
  + 模块系统
    * 在 Node 中没有全局作用域的概念
    * 在 Node 中，只能通过 require 方法来加载执行多个 JavaScript 脚本文件
    * require 加载只能是执行其中的代码，文件与文件之间由于是模块作用域，所以不会有污染的问题
      - 模块完全是封闭的
      - 外部无法访问内部
      - 内部也无法访问外部
    * 模块作用域固然带来了一些好处，可以加载执行多个文件，可以完全避免变量命名冲突污染的问题
    * 但是某些情况下，模块与模块是需要进行通信的
    * 在每个模块中，都提供了一个对象：`exports`
    * 该对象默认是一个空对象
    * 你要做的就是把需要被外部访问使用的成员手动的挂载到 `exports` 接口对象中
    * 然后谁来 `require` 这个模块，谁就可以得到模块内部的 `exports` 接口对象
    * 还有其它的一些规则，具体后面讲，以及如何在项目中去使用这种编程方式，会通过后面的案例来处理
  + 核心模块
    * 核心模块是由 Node 提供的一个个的具名的模块，它们都有自己特殊的名称标识，例如
      - fs 文件操作模块
      - http 网络服务构建模块
      - os 操作系统信息模块
      - path 路径处理模块
      - 。。。。
    * 所有核心模块在使用的时候都必须手动的先使用 `require` 方法来加载，然后才可以使用，例如：
      - `var fs = require('fs')`
- http
  + require
  + 端口号
    * ip 地址定位计算机
    * 端口号定位具体的应用程序
  + Content-Type
    * 服务器最好把每次响应的数据是什么内容类型都告诉客户端，而且要正确的告诉
    * 不同的资源对应的 Content-Type 是不一样，具体参照：http://tool.oschina.net/commons
    * 对于文本类型的数据，最好都加上编码，目的是为了防止中文解析乱码问题
  + 通过网络发送文件
    * 发送的并不是文件，本质上来讲发送是文件的内容
    * 当浏览器收到服务器响应内容之后，就会根据你的 Content-Type 进行对应的解析处理

- 模块系统
- Node 中的其它的核心模块
- 做一个小管理系统：
  + CRUD
- Express Web 开发框架
  + `npm install express`

## 三、Node.js

### 1.HelloWorld

```javascript
var foo = 'HelloWorld node.js Jasonandy'
console.log(foo)
//node.js print  HelloWorld node.js Jasonandy
```

### 2.Node.js没有BOM和DOM

```javascript
// 在 Node 中，采用 EcmaScript 进行编码
// 没有 BOM、DOM
// 和浏览器中的 JavaScript 不一样
console.log(window)
console.log(document)
```

### 3.Node.js 对文件的处理

```javascript
/*浏览器中的 JavaScript 是没有文件操作的能力的
但是 Node 中的 JavaScript 具有文件操作的能力
NODE 面向的是服务端

面向模块化编程

fs 是 file-system 的简写，就是文件系统的意思.
在 Node 中如果想要进行文件操作，就必须引入 fs 这个核心模块
在 fs 这个核心模块中，就提供了所有的文件操作相关的 API
例如：fs.readFile 就是用来读取文件的*/
var fs = require('fs')
/*
1. 使用 require 方法加载 fs 核心模块
2. 读取文件
   第一个参数就是要读取的文件路径
   第二个参数是一个回调函数
       成功
         data 数据
         error null
       失败
         data undefined没有数据
         error 错误对象*/
fs.readFile('./data/test.txt', function (error, data) {
  /*<Buffer 68 65 6c 6c 6f 20 6e 6f 64 65 6a 73 0d 0a>
  文件中存储的其实都是二进制数据 0 1
  这里为什么看到的不是 0 和 1 呢？原因是二进制转为 16 进制了
  但是无论是二进制01还是16进制，人类都不认识
  所以我们可以通过 toString 方法把其转为我们能认识的字符
  console.log(data)
  console.log(error)
  console.log(data)
  在这里就可以通过判断 error 来确认是否有错误发生*/
  if (error) {
    console.log('没有该文件or读异常！')
  } else {
    console.log(data.toString())
  }
})
```

### 4.浏览器不认识Node.js

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <!-- <script src="00-helloworld.js"></script> -->
  <!-- <script src="01-没有bom和dom.js"></script> -->
  <script src="test.node.js"></script>
  <!-- 这样其实是加载不出来的,浏览器无法识别node.js方式的 没有BOM DOM -->
</body>
</html>
```

### 5.Node.js对文件读写操作

```javascript
var fs = require('fs')

/*$.ajax({
  ...
  success: function (data) {
    
  }
})

第一个参数：文件路径
第二个参数：文件内容
第三个参数：回调函数
   error
   成功：
     文件写入成功
     error 是 null
   失败：
     文件写入失败
     error 就是错误对象*/
fs.writeFile('./data/你好.md', 'readNode.js', function (error) {
  // console.log('文件写入成功')
  // console.log(error)
  if (error) {
    console.log('写入失败')
  } else {
    console.log('写入成功了')
  }
})
```

### 6.Node.js Http模块

```javascript
/*接下来，我们要干一件使用 Node 很有成就感的一件事儿
你可以使用 Node 非常轻松的构建一个 Web 服务器
在 Node 中专门提供了一个核心模块：http
http 这个模块的职责就是帮你创建编写服务器的

1. 加载 http 核心模块*/
var http = require('http')

/*2. 使用 http.createServer() 方法创建一个 Web 服务器返回一个 Server 实例*/
var server = http.createServer()

/*3. 服务器要干嘛？
   提供服务：对 数据的服务
   发请求
   接收请求
   处理请求
   给个反馈（发送响应）
   注册 request 请求事件
   当客户端请求过来，就会自动触发服务器的 request 请求事件，然后执行第二个参数：回调处理函数*/
server.on('request', function () {//绑定请求事件
  console.log('收到客户端的请求了')
})

// 4. 绑定端口号，启动服务器
server.listen(3000, function () {
  console.log('Server starting ...')
  console.log('服务器启动成功了，可以通过 http://127.0.0.1:3000/访问')
})
```

### 7.Node.js http-Res 

```javascript
var http = require('http')

var server = http.createServer()
/*request 请求事件处理函数，需要接收两个参数：
   Request 请求对象
       请求对象可以用来获取客户端的一些请求信息，例如请求路径
   Response 响应对象
       响应对象可以用来给客户端发送响应消息*/
server.on('request', function (request, response) {
/*  http://127.0.0.1:3000/ /
  http://127.0.0.1:3000/a /a
  http://127.0.0.1:3000/foo/b /foo/b*/
  console.log('收到客户端的请求了，请求路径是：' + request.url)

  // response 对象有一个方法：write 可以用来给客户端发送响应数据  将数据写出去
  // write 可以使用多次，但是最后一定要使用 end 来结束响应，否则客户端会一直等待
  response.write('hello')
  response.write(' nodejs')

  // 告诉客户端，我的话说完了，你可以呈递给用户了,前端开始渲染
  response.end()

  /*由于现在我们的服务器的能力还非常的弱，无论是什么请求，都只能响应 hello nodejs
  思考：
   我希望当请求不同的路径的时候响应不同的结果
   例如：
   / index
   /login 登陆
   /register 注册
   /haha 哈哈哈*/
})

server.listen(3000, function () {
  console.log('服务器启动成功了，可以通过 http://localhost:3000/ 来进行访问')
})
```

### 8.Node.js http-url-res

```javascript
//引入模块
var http = require('http')

// 1. 创建 Server
var server = http.createServer()

// 2. 监听 request 请求事件，设置请求处理函数
server.on('request', function (req, res) {
  console.log('收到请求了，请求路径是：' + req.url)
  //socket - remoteAddress remotePort
  console.log('请求我的客户端的地址是：', req.socket.remoteAddress, req.socket.remotePort)

  // res.write('hello')
  // res.write(' world')
  // res.end()

  // 上面的方式比较麻烦，推荐使用更简单的方式，直接 end 的同时发送响应数据
  // res.end('hello nodejs')

  // 根据不同的请求路径发送不同的响应结果
  // 1. 获取请求路径
  //    req.url 获取到的是端口号之后的那一部分路径
  //    也就是说所有的 url 都是以 / 开头的
  // 2. 判断路径处理响应

  var url = req.url

  if (url === '/') {
    res.end('index page')
  } else if (url === '/login') {
    res.end('login page')
  } else if (url === '/products') {
    var products = [{
        name: '华为X',
        price: 8888
      },
      {
        name: 'OppO X',
        price: 5000
      },
      {
        name: 'VIVO X',
        price: 1999
      }
    ]
    // 响应内容只能是二进制数据或者字符串
    //  数字
    //  对象
    //  数组
    //  布尔值
    res.end(JSON.stringify(products))//序列化 但是还是有乱码
  } else {
    res.end('404 Not Found.')
  }
})

// 3. 绑定端口号，启动服务
server.listen(3000, function () {
  console.log('服务器启动成功，可以访问了。。。')
})

```

### 9.Node.js 常见的核心模块

```javascript
//http模块
var http = require('http')

// 用来获取机器信息的
var os = require('os')

// 用来操作路径的
var path = require('path')

// 获取当前机器的 CPU 信息
console.log(os.cpus())

// memory 内存
console.log(os.totalmem())

// 获取一个路径中的扩展名部分
// extname extension name
console.log(path.extname('d:/node/hello.txt'))

//node 执行一个文件 那怎么处理  引入代码 模块化 

```

### 10.Node.js简单的模块化

#### 1）自定义模块

模块一

```javascript
//module.b 
/*require 方法有两个作用：
   1. 加载文件模块并执行里面的代码
   2. 拿到被加载文件模块导出的接口对象
   
   在每个文件模块中都提供了一个对象：exports
   exports 默认是一个空对象
   把所有需要被外部访问的成员挂载到这个 exports 对象中*/
var bExports = require('./module.b.js')  //加载到模块b了
var fs = require('fs')

console.log(bExports.foo)

console.log(bExports.add(10, 30))

console.log(bExports.age)

bExports.readFile('./moudule.a.js')
//exports 默认是一个空对象  需要被外部访问的成员挂载到这个 exports 对象中

fs.readFile('./moudule.a.js', function (err, data) {
  if (err) {
    console.log('读取文件失败')
  } else {
    console.log(data.toString())
  }
})
```

模块二

```javascript
//module.b.js
var foo = 'node.js module'
console.log(exports)
//exports 暴露成员
exports.foo = 'hello'
exports.add = function (x, y) {
  return x + y
}

exports.readFile = function (path, callback) {
  console.log('文件路径：', path)
}

var age = 2018-1194
exports.age = age

function add(x, y) {
  return x - y
}
```

#### 2）加载模块

```javascript
//主模块 加载别的模块
/*require 方法有两个作用：
   1. 加载文件模块并执行里面的代码
   2. 拿到被加载文件模块导出的接口对象
   
   在每个文件模块中都提供了一个对象：exports
   exports 默认是一个空对象
   你要做的就是把所有需要被外部访问的成员挂载到这个 exports 对象中
   
   在 Node 中，模块有三种：
   具名的核心模块，例如 fs、http
   用户自己编写的文件模块
     相对路径必须加 ./
     可以省略后缀名 （省略.js）
     相对路径中的 ./ 不能省略，否则报错
     
   在 Node 中，没有[全局作用域]，只有模块作用域  -- 可以理解为文件作用域名
     外部访问不到内部
     内部也访问不到外部
     默认都是封闭的
   既然是模块作用域，那如何让模块与模块之间进行通信
   有时候，我们加载文件模块的目的不是为了简简单单的执行里面的代码，更重要是为了使用里面的某个成员
   
   exports -- 暴露成员
*/

var bExports = require('自定义的模块')
var fs = require('fs')
console.log(bExports.foo)

console.log(bExports.add(10, 30))

console.log(bExports.age)

bExports.readFile('test.js')

fs.readFile('./test.js', function (err, data) {
  if (err) {
    console.log('读取文件失败')
  } else {
    console.log(data.toString())
  }
})
```

#### 2）导出成员

```javascript
/*require 方法有两个作用：
   1. 加载文件模块并执行里面的代码
   2. 拿到被加载文件模块导出的接口对象  (可以拿到 文件模块导出的接口接口对象exports )
   
   var bExports = require('自定义的模块') 
   bExports： 默认是一个空的对象
   可以把需要对外的function挂载上去  方便外面的调用
   
   在每个文件模块中都提供了一个对象：exports
   exports 默认是一个空对象
   你要做的就是把所有需要被外部访问的成员挂载到这个 exports 对象中
   
   在 Node 中，模块有三种：
   具名的核心模块，例如 fs、http
   用户自己编写的文件模块
     相对路径必须加 ./
     可以省略后缀名 （省略.js）
     相对路径中的 ./ 不能省略，否则报错
     
   在 Node 中，没有[全局作用域]，只有模块作用域  -- 可以理解为文件作用域名
     外部访问不到内部
     内部也访问不到外部
     默认都是封闭的
   既然是模块作用域，那如何让模块与模块之间进行通信
   有时候，我们加载文件模块的目的不是为了简简单单的执行里面的代码，更重要是为了使用里面的某个成员
   
   exports -- 暴露成员
*/

//module.b.js
var foo = 'node.js module'
console.log(exports)
//exports 暴露成员
exports.foo = 'hello'
exports.add = function (x, y) {
  return x + y
}

exports.readFile = function (path, callback) {
  console.log('文件路径：', path)
}

var age = 2018-1194
exports.age = age

function add(x, y) {
  return x - y
}
```

### 11.Node.js ip和端口

```javascript
/*ip 地址用来定位计算机
端口号用来定位具体的应用程序
所有需要联网通信的应用程序都会占用一个端口号*/

var http = require('http')

var server = http.createServer()

// 2. 监听 request 请求事件，设置请求处理函数
server.on('request', function (req, res) {
  console.log('收到请求了，请求路径是：' + req.url)
  console.log('请求我的客户端的地址是：', req.socket.remoteAddress, req.socket.remotePort)

  res.end('hello nodejs')
})

server.listen(5000, function () {
  console.log('服务器启动成功，可以访问了。。。')
})
```

### 12.Node.js http-content-type

```javascript
/*require ip 端口号*/
var http = require('http')
var server = http.createServer()
server.on('request', function (req, res) {
  /*在服务端默认发送的数据，其实是 utf8 编码的内容
  但是浏览器不知道你是 utf8 编码的内容
  浏览器在不知道服务器响应内容的编码的情况下会按照当前操作系统的默认编码去解析
  中文操作系统默认是 gbk
  解决方法就是正确的告诉浏览器我给你发送的内容是什么编码的
  在 http 协议中，Content-Type 就是用来告知对方我给你发送的数据内容是什么类型
  res.setHeader('Content-Type', 'text/plain; charset=utf-8')
  res.end('hello 世界')*/
  var url = req.url
  if (url === '/plain') {
    // text/plain 就是普通文本
    res.setHeader('Content-Type', 'text/plain; charset=utf-8')
    res.end('hello 世界')
  } else if (url === '/html') {
    // 如果你发送的是 html 格式的字符串，则也要告诉浏览器我给你发送是 text/html 格式的内容
    res.setHeader('Content-Type', 'text/html; charset=utf-8')
    res.end('<p>hello html <a href="">点我</a></p>')
  }
})

server.listen(3000, function () {
  console.log('Server is running...')
})
```

### 13.Node.js fs 实现数据交互

```javascript
/*1. 结合 fs 发送文件中的数据
2. Content-Type
   http://tool.oschina.net/commons
   不同的资源对应的 Content-Type 是不一样的
   图片不需要指定编码
   一般只为字符数据才指定编码*/
var http = require('http')
var fs = require('fs')

var server = http.createServer()

server.on('request', function (req, res) {
  // / index.html
  var url = req.url

  if (url === '/') {
    // 肯定不这么干
    // res.end('<!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"><title>Document</title></head><body><h1>首页</h1></body>/html>')

    // 我们要发送的还是在文件中的内容
    fs.readFile('./resource/index.html', function (err, data) {
      if (err) {
        res.setHeader('Content-Type', 'text/plain; charset=utf-8')
        res.end('文件读取失败，请稍后重试！')
      } else {
        // data 默认是二进制数据，可以通过 .toString 转为咱们能识别的字符串
        // res.end() 支持两种数据类型，一种是二进制，一种是字符串
        res.setHeader('Content-Type', 'text/html; charset=utf-8')
        res.end(data)
      }
    })
  } else if (url === '/xiaoming') {
    // url：统一资源定位符
    // 一个 url 最终其实是要对应到一个资源的
    fs.readFile('./resource/ab2.jpg', function (err, data) {
      if (err) {
        res.setHeader('Content-Type', 'text/plain; charset=utf-8')
        res.end('文件读取失败，请稍后重试！')
      } else {
       /* data 默认是二进制数据，可以通过 .toString 转为咱们能识别的字符串
        res.end() 支持两种数据类型，一种是二进制，一种是字符串
        图片就不需要指定编码了，因为我们常说的编码一般指的是：字符编码*/
        res.setHeader('Content-Type', 'image/jpeg')
        res.end(data)
      }
    })
  }
})

server.listen(3000, function () {
  console.log('Server is running...')
})

```

