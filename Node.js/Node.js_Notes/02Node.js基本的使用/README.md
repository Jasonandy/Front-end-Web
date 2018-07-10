# fNode.js 模块及相关原理

## 一、模块系统

```javascript
//modulue.a.js
var foo = 'bar'

function add(x, y) {
  return x + y
}

// 这种方式不行。
// exports = add

// 如果一个模块需要直接导出某个成员，而非挂载的方式
// 那这个时候必须使用下面这种方式 [module]  直接赋值过去
module.exports = 'hello' //得到的字符串串

// Function   会覆盖前者 
module.exports = function (x, y) {
  return x + y
}

//导出多个函数or成员 
module.exports = {
  add: function () {
    return x + y
  },
  str: 'hello'，
  print: function(){
  	console.log('jasonandy@hotmail.com')
  }
}

// 你可以认为在每个模块的最后 return 了这个 exports

// 只能得到我想要给你的成员
// 这样做的目的是为了解决变量命名冲突的问题
// exports.add = add

// exports 是一个对象
// 我们可以通过多次为这个对象添加成员实现对外导出多个内部成员

// exports.str = 'hello'

// 现在我有一个需求：
// 我希望加载得到直接就是一个：
//  方法
//  字符串
//  数字
//  数组
```

对模块的调用

```javascript
/*默认得到的是对象
使用对象中的成员必须 . 点儿某个成员来访问
有时候，对于一个模块，我仅仅就是希望导出一个方法就可以了*/
var fooExports = require('./modulue.a.js')

// ReferenceError: foo is not defined
// console.log(foo)

console.log(fooExports)
```

## 二、模块原理

```javascript
/*在 Node 中，每个模块内部都有一个自己的 module 对象
该 module 对象中，有一个成员叫：exports 也是一个对象
也就是说如果你需要对外导出成员，只需要把导出的成员挂载到 module.exports 中

我们发现，每次导出接口成员的时候都通过 module.exports.xxx = xxx 的方式很麻烦，点儿的太多了
所以，Node 为了简化你的操作，专门提供了一个变量：exports 等于 module.exports

//Node.js 提供了一个变量 exports == module.exports [简化]   暴露的时候是可以直接用的

var module = {
  //exports 也是一个对象 默认是一个空对象
  exports: {
    foo: 'bar',
    add: function
  }
  //最后有一个隐形的return module.exports 
}

也就是说在模块中还有这么一句代码
var exports = module.exports

module.exports.foo = 'bar'

module.exports.add = function (x, y) {
  return x + y
}

两者一致，那就说明，我可以使用任意一方来导出内部成员
console.log(exports === module.exports)

exports.foo = 'bar'
module.exports.add = function (x, y) {
  return x + y
}

当一个模块需要导出单个成员的时候
直接给 exports 赋值是不管用的

exports.a = 123

exports = {}
exports.foo = 'bar'

module.exports.b = 456

给 exports 赋值会断开和 module.exports 之间的引用
同理，给 module.exports 重新赋值也会断开

这里导致 exports !== module.exports
module.exports = {
  foo: 'bar'
}

// 但是这里又重新建立两者的引用关系
exports = module.exports

exports.foo = 'hello'

{foo: bar}*/
exports.foo = 'bar'


// {foo: bar, a: 123}
module.exports.a = 123

// exports !== module.exports
// 最终 return 的是 module.exports
// 所以无论你 exports 中的成员是什么都没用
exports = {
  a: 456
}

// {foo: 'haha', a: 123}
module.exports.foo = 'haha'

// 没关系，混淆你的
exports.c = 456

// 重新建立了和 module.exports 之间的引用关系了
exports = module.exports

// 由于在上面建立了引用关系，所以这里是生效的
// {foo: 'haha', a: 789}
exports.a = 789

// 前面再牛逼，在这里都全部推翻了，重新赋值
// 最终得到的是 Function
module.exports = function () {
  console.log('hello')
}

/*真正去使用的时候：
   导出多个成员：exports.xxx = xxx
   导出多个成员也可以：module.exports = {
                       }
   导出单个成员：module.exports


谁来 require 我，谁就得到 module.exports
默认在代码的最后有一句：
一定要记住，最后 return 的是 module.exports
不是 exports
所以你给 exports 重新赋值不管用，
return module.exports*/

```

```javascript
var fooExports = require('./foo')

console.log(fooExports)


/*如果你实在分不清楚 exports 和 module.exports
你可以选择忘记 exports
而只使用 module.exports 也没问题

[module.exports] 默认用一个，尽量避免混合使用.

module.exports.xxx = xxx
moudle.exports = {}*/
```

## 三、require 加载顺序

```javascript
console.log('a.js 被加载了')
var fn = require('./b')

console.log(fn)
```

```javascript
console.log('b.js 被加载了')

module.exports = function () {
  console.log('hello bbb')
}
```

```javascript
require('./a')

/*优先从缓存加载
由于 在 a 中已经加载过 b 了
所以这里不会重复加载
可以拿到其中的接口对象，但是不会重复执行里面的代码
这样做的目的是为了避免重复加载，提高模块加载效率

不会重复加载 缓存中加载  不会重复执行代码
*/
var fn = require('./b')

console.log(fn)
```

## 四、require 标识符

```javascript
/*如果是非路径形式的模块标识【os,file,第三方，自定义】
路径形式的模块：
 ./ 当前目录，不可省略  [当前目录]
 ../ 上一级目录，不可省略
 /xxx 几乎不用【表示的当前的文件模块的Root目录】
 d:/a/foo.js 几乎不用【绝对路径】 可移植性不好
 首位的 / 在这里表示的是当前文件模块所属磁盘根路径
 .js 后缀名可以省略
require('./foo.js')  == require('./foo')

核心模块的本质也是文件
核心模块文件已经被编译到了二进制文件中了，我们只需要按照名字来加载就可以了
require('fs')
require('http')
//github 可以看到lib里面的源码 fs.js http.js etc..

第三方模块
凡是第三方模块都必须通过 npm 来下载   node package manager [rpm]
使用的时候就可以通过 require('包名') 的方式来进行加载才可以使用
不可能有任何一个第三方包和核心模块的名字是一样的
既不是核心模块、也不是路径形式的模块
   先找到当前文件所处目录中的 node_modules 目录
   node_modules/art-template
   node_modules/art-template/package.json 文件
   node_modules/art-template/package.json 文件中的 main 属性
   main 属性中就记录了 art-template 的入口模块
   然后加载使用这个第三方包
   实际上最终加载的还是文件

   如果 package.json 文件不存在或者 main 指定的入口模块是也没有
   则 node 会自动找该目录下的 index.js
   也就是说 index.js 会作为一个默认备选项
   [package.json  的main指向的 如果没有默认的index.js]
   如果以上所有任何一个条件都不成立，则会进入上一级目录中的 node_modules 目录查找
   如果上一级还没有，则继续往上上一级查找
   。。。
   如果直到当前磁盘根目录还找不到，最后报错：/a/b/c/... 一层一层查找 
     can not find module xxx
var template = require('art-template')

注意：我们一个项目有且只有一个 node_modules，放在项目根目录中，这样的话项目中所有的子目录中的代码都可以加载到第三方包
不会出现有多个 node_modules
模块查找机制
   1.优先从缓存加载
   2.核心模块
   3.路径形式的文件模块
   4.第三方模块
     node_modules/art-template/
     node_modules/art-template/package.json
     node_modules/art-template/package.json main
     index.js 备选项
     进入上一级目录找 node_modules
     按照这个规则依次往上找，直到磁盘根目录还找不到，最后报错：Can not find moudle xxx
   一个项目有且仅有一个 node_modules 而且是存放到项目的根目录*/

require('a')

//foot.js
console.log('foo 文件模块被加载了')
```

## 五、Express

```javascript
//express 
原生的http在某些方面的表现不足以应对我们的开发需求，用框架来加快我们的开发效率

var express = require('express');

var app = express();

 app.get('/',function (req , res) {
     //res.send('HelloWorld!');
     //res.send('yeap');
     res.send('你好，世界！');
 });
  app.get('/app',function (req , res) {
     //res.send('HelloWorld!');
     //res.send('yeap');
     res.send('你好，世界！app.');
 });

// express 开放资源
// 模板引擎等 

//公开指定目录 http://localhost:3000/public/js/test.js
app.use('/public',express.static('./public/'))

app.use('/static',express.static('./static/'))

 app.listen(3000,function(){
    console.log('Server is starting on port 3000');
 })


```

## 六、npm

```javascript
//npm
npm i xxx --save 

npm install 会检测到你的package [node_moudlue]丢失也没有问题

npmjs.com   类似yum的repo仓库源  docker源 maven源

npm命令行 工具 
	npm也有版本概念
     npm insatll --global npm 自己升级自己
     
常用的命令
	npm init  
    	npm init -y 快速生成
     npm install         //dependencies 依赖包安装好
     npm install xxx    //只下载
     npm install xxx --save  //下载保存
    
     npm uninstall
     	只删除 依赖依然会保存
        
     npm unistall --save xxx 
     	删除的同时会改变依赖信息
        
     npm --help 
     	查看命令的使用帮助
     
     npm uninstall --help
     //可以查看使用的帮助
     
     解决npm被墙的问题 服务器在国外  访问不稳定 
     使用阿里的镜像（cnpm）
     
     npm insatll --global cnpm [taobao]

    npm insatall jquery --registry=http://.../taobao.org
    可以加入到配置文件
    npm cofig set registry https://xxx.npm.taobao/org

	npm config list //查看信息
```

## 七、package.json

```javascript
//npm
包的描述文件 可以理解为产品的说明书

main : index.js 入口

npm init 可以设置package
 
//描述信息 

//dependencies

建议每个项目的根目录都有一个package.json
npm i xxx --save 建议都保存一下

```

## 八、要点小结

- jQuery 的 each 和 原生的 JavaScript 方法 forEach
  + EcmaScript 5 提供的
    * 不兼容 IE 8
  + jQuery 的 each 由 jQuery 这个第三方库提供
    * jQuery 2 以下的版本是兼容 IE 8 的
    * 它的 each 方法主要用来遍历 jQuery 实例对象（伪数组）
    * 同时它也可以作为低版本浏览器中 forEach 替代品
    * jQuery 的实例对象不能使用 forEach 方法，如果想要使用必须转为数组才可以使用
    * `[].slice.call(jQuery实例对象)`
- 模块中导出多个成员和导出单个成员
- 301 和 302 状态码区别
  + 301 永久重定向，浏览器会记住
  + 302 临时重定向
- exports 和 module.exports 的区别
  + 每个模块中都有一个 module 对象
  + module 对象中有一个 exports 对象
  + 我们可以把需要导出的成员都挂载到 module.exports 接口对象中
  + 也就是：`moudle.exports.xxx = xxx` 的方式
  + 但是每次都 `moudle.exports.xxx = xxx` 很麻烦，点儿的太多了
  + 所以 Node 为了你方便，同时在每一个模块中都提供了一个成员叫：`exports`
  + `exports === module.exports` 结果为  `true`s
  + 所以对于：`moudle.exports.xxx = xxx` 的方式 完全可以：`expots.xxx = xxx`
  + 当一个模块需要导出单个成员的时候，这个时候必须使用：`module.exports = xxx` 的方式
  + 不要使用 `exports = xxx` 不管用
  + 因为每个模块最终向外 `return` 的是 `module.exports`
  + 而 `exports` 只是 `module.exports` 的一个引用
  + 所以即便你为 `exports = xx` 重新赋值，也不会影响 `module.exports`
  + 但是有一种赋值方式比较特殊：`exports = module.exports` 这个用来重新建立引用关系的
  + 之所以让大家明白这个道理，是希望可以更灵活的去用它
- Node 是一个比肩 Java、PHP 的一个平台
  + JavaScript 既能写前端也能写服务端

```javascript
moudle.exports = {
  a: 123
}

// 重新建立 exports 和 module.exports 之间的引用关系
exports = module.exports

exports.foo = 'bar'
```

```javascript
Array.prototype.mySlice = function () {
  var start = 0
  var end = this.length
  if (arguments.length === 1) {
    start = arguments[0]
  } else if (arguments.length === 2) {
    start = arguments[0]
    end = arguments[1]
  }
  var tmp = []
  for (var i = start; i < end; i++) {
    // fakeArr[0]
    // fakeArr[1]
    // fakeArr[2]
    tmp.push(this[i])
  }
  return tmp
}

var fakeArr = {
  0: 'abc',
  1: 'efg',
  2: 'haha',
  length: 3
}

// 所以你就得到了真正的数组。 
[].mySlice.call(fakeArr)
```

- jQuery 的 each 和 原生的 JavaScript 方法 forEach
- 301 和 302 的区别
- 模块中导出单个成员和导出多个成员的方式
- module.exports 和 exports 的区别
- require 方法加载规则
  + 优先从缓存加载
  + 核心模块
  + 路径形式的模块
  + 第三方模块
    * node_modules
- package.json 包描述文件
  + dependencies 选项的作用
- npm 常用命令
- Express 基本使用

