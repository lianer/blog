# bodyParser组件的研究
> 接触nodejs已有一段时间了，但最近才开始落实项目，于是使用express应用生成器生成了一个应用，在开发中发现，ajax提交的数据无法被express正确的解析，莫名其妙的一个坑，困了我许久。


bodyParser中间件用来解析http请求体，是express默认使用的中间件之一。


使用express应用生成器生成一个网站，发现它默认已经开启了json与urlencoded的解析功能，除了这两个，bodyParser还支持对text、raw的解析。
```javascript
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```

顾名思义，json是用来解析json数据格式的。urlencoded则是用来解析我们通常的form表单提交的数据，也就是请求头中的描述为： `Content-Type: application/x-www-form-urlencoded`

## 常见的四种Content-Type类型
- `application/x-www-form-urlencoded` 常见的form提交
- `multipart/form-data` 文件提交
- `application/json` 提交json格式的数据
- `text/xml` 提交xml格式的数据

## 详细解读 urlencoded
urlencoded模块用于解析req.body的数据，解析成功后覆盖原来的req.body，如果解析失败则为 `{}`。该模块有一个属性extended，官方介绍如下：

The extended option allows to choose between parsing the URL-encoded data with the querystring library (when false) or the qs library (when true). Defaults to true, but using the default has been deprecated.

大致的意思就是：extended选项允许配置使用querystring(false)或qs(true)来解析数据，默认值是true，但这已经是不被赞成的了。

querystring就是nodejs内建的对象之一，用来字符串化对象或解析字符串。如
```javascript
querystring.parse("name=henry&age=30") => { name: 'henry', age: '30' }
```
那么，既然他已经能完成对urlencode的解析了，为什么还需要qs？qs又是什么？
