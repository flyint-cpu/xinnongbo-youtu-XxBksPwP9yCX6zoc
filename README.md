
后端开发，我们对于Api接口调试测试大致有以下方法：单元测试、Swagger、Postman。


但是每种方式也都有其局限性，几年前使用Visual Studio Code开发过一段时间，接触了REST Client扩展工具印象特别深刻，简单、轻量、可编码、与开发工具无缝衔接，整体效率相当高。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004323441-1883367057.png)


再此之后我一直在关注Visual Studio是否有类似的工具，直到最近发现Visual Studio 2022版本17\.8开始支持类似REST Client扩展工具相关功能了，虽然功能还不够完善但是也基本够用了。


今天和大家分享怎么通过.http文件便捷调试测试Api接口。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004331056-1588066860.png)


# ***01***、.http文件创建方式


.http文件有两种创建方式：其一为通过添加文件，其二为通过终结点资源管理器生成。


## 1、添加文件方式


就像平时添加类文件一样，通过选择类库右击选择添加，选择新建项，然后找到HTTP文件选项，可以修改名称最后点击添加按钮即可。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004338239-917693352.png)


## 2、终结点资源管理器生成方式


首先选择视图菜单，找到其他窗口，然后找到终结点资源管理器并点击，即可打开。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004346849-1458671631.png)


打开后效果如下：


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004354797-1107668610.png)


然后我们可以任意选择一个接口，右击按钮并点击生成请求，即可自动创建.http文件并自动生成当前接口的默认请求示例，如下图：


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004401843-445464046.png)


# ***02***、.http 文件语法


## 1、请求


HTTP请求格式为\[HTTPMethod URL HTTPVersion]。


**HTTPMethod：** 表示HTTP方法包括增删改查四大常见方法：GET、POST、PUT、DELETE，以及一些其他HTTP方法OPTIONS、HEAD、PATCH、TRACE、CONNECT；


**URL：** 表示发送请求的URL，即请求目标URL，像正常方法URL一样可以包含查询字符串参数；


**HTTPVersion：** 此项为可选项，是指应用的HTTP版本，即 HTTP/1\.1、HTTP/2 或 HTTP/3。


当然一个.http文件中可以包含多个请求，可以通过\#\#\#作为分隔符把多个请求分开



```
POST https://localhost:5137/orders

###

GET https://localhost:5137/orders?id=98006

###

DELETE https://localhost:5137/orders HTTP/3

###

```

## 2、请求头、请求体


在实际请求中，我们不单单要指定请求方法，请求URL，还需要指定请求头以及请求体。


常见的请求有请求内容类型、响应内容类型、编码方式、缓存控制、内容类型、身份验证、跨域请求等等。


请求头格式为\[HeaderName: Value]，一个请求头类型占一行，可以有多个请求头，并且每个请求头之间不能有空白行，请求头和请求行之间也不能有空白行。



```
GET https://localhost:5137/orders
Accept: application/json, text/html

###

GET https://localhost:5137/orders
Cache-Control: max-age=604800
Age: 100

###

```

请求体是指HTTP请求中携带的实际数据，在请求行或者请求头后空白行之后添加，示例如下：



```
GET https://localhost:5137/orders
Content-Type: application/json

{
  "id": "897",
  "date": "2024-12-24",
  "price": 5,
  "priceF": 2,
  "name": "小红",
  "status": "Pending"
}

###

```

## 3、注释、变量


注释是以\#或//开头的行，可以加强代码的可读性。


变量是以@开头的行，其语法格式为\[@VariableName\=Value]，定义好变量后可以通过双大括号{{ VariableName }}来使用变量，同时也可以使用已经定义好的变量来定义新的变量。



```
@hostname=localhost
@port=5137
@host={{hostname}}:{{port}}
GET https://{{host}}/orders?id=98006

```

## 4、环境文件


在实际开发过程中，针对开发环境、测试环境，甚至生产环境，我们需要对同一个变量提供不同的值，比如不同环境首先域名就不同，其次针对不同环境的测试数据也不同。


这时候我们就可以使用环境文件，我们可以在.http文件所在的目录中或者其父目录中创建名为http\-client.env.json的文件。如下代码，我们创建了开发环境和生产环境两个不同的域名。



```
{
  "dev": {
    "HostAddress": "http://localhost:5137"
  },
  "pro": {
    "HostAddress": "http://localhost:8888"
  }
}

```

此时我们可以在.http文件窗口右上角进行切换不同环境。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004422374-2109635632.png)


下面我们选择pro环境进行测试一下，结果如下。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004429833-1813936632.png)


可以看到此时域名读取了正式环境域名。


此时我们是可以把环境文件提交到代码库中和团队共享测试变量，那如果有些敏感的数据我们并不想提交到代码库和别人分享要怎么办呢？


我们可以在环境文件同级目录中创建http\-client.env.json.user文件，它和环境文件编写规则完全一样，只是优先级比环境文件优先级更高。当然我们需要在代码管理的忽略文件.gitignore排查.user后缀的文件，防止其被意外提交至代码块。


# ***03***、身份验证


可以说我们的每个后端接口都有相关认证授权，可能使用Jwt，OAuth令牌，API密钥，用户密码等方式，这就导致我们平时测试的时候，需要先登录，然后拿到相关的认证凭证，再去调相关的接口。


这意味这我们面临一种情况，调用B接口需要依赖A接口的返回结果。首先答案很明确可以做到，要怎么做呢？


我们可以在A接口请求上面使用以下语法\[\# @name VariableName]，来定义承载接口返回结果变量。


下面我们实现一个登录接口，直接返回Jwt凭证即token字符串，然后使用这个token请求查询订单接口。



```
//登录
# @name login
POST {{Web_HostAddress}}/login
Accept: application/json

###

//查询订单
@id=897
GET {{Web_HostAddress}}/orders?id={{id}}
Authorization: Bearer {{login.response.body.$.[0]}}

###

```

需要注意的是，因为我们登录即可是直接返回的字符串，所以这里使用的是login.response.body.$.\[0]，如果我们返回的是对象并且token是赋值在token字段上的，则应该使用login.response.body.$.token。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241225004438708-2001342603.png)


并且当我们点击查询订单接口请求上面的调试时，即使我们程序没有运行起来，它也会自动运行，并且也会自动去执行登录接口，拿到它所需的token，然后运行自身。


而发送请求按钮就必须要求程序运行起来以后才能生效。


到这里.http文件使用就介绍完了，可以说基本够用了，当然还有很多地方需要完善的，比如说上传文件，终结点资源管理器不能直接自动生成请求体结构等等，相信要不了多久这些功能就会完善起来的，安心等待即可。


***注***：测试方法代码以及示例源码都已经上传至代码库，有兴趣的可以看看。[https://gitee.com/hugogoos/Planner](https://github.com)


 本博客参考[slowerssr加速器](https://slowerss.com)。转载请注明出处！
