# bodyParser中间件的研究
接触nodejs已有一段时间了，但最近才开始落实项目，于是使用express应用生成器生成了一个应用。开发过程中发现ajax提交的数据无法被express正确的解析，主要的情况是这样的：
```javascript
// 浏览器端post一个对象
$.ajax({
    url: "/save",
    type: "post",
    data: {
        name: "henry",
        age: 30,
        hobby: [ "sport", "coding" ]
    }
});

// express接收这个对象
router.post("/save", function (req, res, next) {
    console.log(req.body); // => { 'info[name]': 'henry','info[age]': '30','hobby[1]': 'sport','hobby[2]': 'coding' }
});
```
显然这样的解析结果是不能直接拿来用的，莫名其妙的一个坑，困了我许久。


## bodyParser中间件
bodyParser中间件用来解析http请求体，是express默认使用的中间件之一。


使用express应用生成器生成一个网站，它默认已经使用了 `bodyParser.json` 与 `bodyParser.urlencoded` 的解析功能，除了这两个，bodyParser还支持对text、raw的解析。
```javascript
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```

顾名思义，bodyParser.json是用来解析json数据格式的。bodyParser.urlencoded则是用来解析我们通常的form表单提交的数据，也就是请求头中包含这样的信息： `Content-Type: application/x-www-form-urlencoded`

**常见的四种Content-Type类型：**

- `application/x-www-form-urlencoded` 常见的form提交
- `multipart/form-data` 文件提交
- `application/json` 提交json格式的数据
- `text/xml` 提交xml格式的数据

## 详细解读 urlencoded
`bodyParser.urlencoded` 模块用于解析req.body的数据，解析成功后覆盖原来的req.body，如果解析失败则为 `{}`。该模块有一个属性extended，官方介绍如下：

The extended option allows to choose between parsing the URL-encoded data with the querystring library (when false) or the qs library (when true). Defaults to true, but using the default has been deprecated.

大致的意思就是：extended选项允许配置使用querystring(false)或qs(true)来解析数据，默认值是true，但这已经是不被赞成的了。

querystring就是nodejs内建的对象之一，用来字符串化对象或解析字符串。如
```javascript
querystring.parse("name=henry&age=30") => { name: 'henry', age: '30' }
```
那么，既然querystring已经能完成对urlencode的解析了，为什么还需要qs？qs又是什么？

## qs介绍
qs是一个querystring的库，在qs的功能基础上，还支持更多的功能并优化了一些安全性。比如，对象解析的支持：
```javascript
// 内建对象 querystring
querystring.parse("info[name]=henry&info[age]=30&hobby[1]=sport&hobby[2]=coding") => 
  { 
    'info[name]': 'henry',
    'info[age]': '30',
    'hobby[1]': 'sport',
    'hobby[2]': 'coding'
  }

// 第三方插件 qs
qs.parse("info[name]=henry&info[age]=30&hobby[1]=sport&hobby[2]=coding") => 
  {
    info: {
      name: 'henry',
      age: '30'
    },
    hobby: [ 'sport', 'coding' ]
  }
```
可以看出，querystring并不能正确的解析复杂对象（多级嵌套），而qs却可以做到。

但是qs也不是万能的，对于多级嵌套的对象，qs只会解析5层嵌套，超出的部分会表现的跟本文头部的那种情况一样；对于数组，qs最大只会解析20个索引，超出的部分将会以键值对的形式解析。

作为一个中间件，qs必须要为性能考虑，才会有如此多的限制，express也默认使用qs来解析请求体。

理论上来说，form表单提交不会有多级嵌套的情况，而urlencoded本身也是form的内容类型，因此，bodyParser.urlencoded不支持多级嵌套也是很合理的设计。

那么，如果我们非要上传一个十分复杂的对象，应该怎么办？

## 解决方案
出现这个问题的根本原因是：我以form的形式去提交了一个json数据。

jquery默认的 `content-Type` 配置的是 `application/x-www-form-urlencoded`，

因此更改ajax请求参数：`contentType: "application/json"`，并将数据转成json提交，问题就解决了。
```javascript
// 浏览器端post一个对象
$.ajax({
    url: "/save",
    type: "post",
    contentType: "application/json",
    data: JSON.stringify({
        name: "henry",
        age: 30,
        hobby: [ "sport", "coding" ]
    })
});

// express接收这个对象
router.post("/save", function (req, res, next) {
    console.log(req.body); // => { name: 'henry', age: 30, hobby: [ 'sport', 'coding' ] }
});
```

## 参考资料
- [body-parser](https://github.com/expressjs/body-parser)
- [qs](https://github.com/ljharb/qs)

**大多时候，我们只知道如何去使用，而不知道为什么这么用。**