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
express应用生成器默认配置了 `{extended: false}`，这个属性在官方的文档中的介绍是这样的：