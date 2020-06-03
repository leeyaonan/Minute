# API网关实现跨域资源共享（CORS）

>https://help.aliyun.com/document_detail/89765.html?spm=5176.11065259.1996646101.searchclickresult.531e46efe7I7WA

# 一.跨域带来的安全问题及浏览器的限制访问

当一个资源从与该资源本身所在的服务器不同的域或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。比如，站点 [http://www.aliyun.com](http://www.aliyun.com/) 的某 HTML 页面通过img的src请求 http://www.alibaba.com/image.jpg 。 网络上的许多页面都会加载来自不同域的CSS样式表，图像和脚本等资源。

出于安全原因，浏览器限制从页面脚本内发起的跨域请求，有些浏览器不会限制跨域请求的发起，但是会将结果拦截了。 这意味着使用这些API的Web应用程序只能加载同一个域下的资源，除非使用CORS机制（Cross-Origin Resource Sharing 跨源资源共享）获取目标服务器的授权来解决这个问题。

[![跨域访问示例图](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200602114729.png)](https://cdn.nlark.com/lark/0/2018/png/18611/1535462794732-ce89de46-bda5-4864-b4d3-9e137912e3e7.png)

上图画的是典型的跨域场景，目前主流浏览器为了用户的安全，都会默认禁止跨域访问，但是主流浏览器都支持W3C推荐了一种跨域资源共享机制（CORS）。服务器端配合浏览器实现CORS机制，可以突破浏览器对跨域资源访问的限制，实现跨域资源请求。

[![主流浏览器都已基本提供对跨域资源共享CORS的支持](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200602114733.png)](https://cdn.nlark.com/lark/0/2018/png/18611/1535464164919-d0dd8021-98d5-4af3-803d-d3468be223e3.png)

# 二.跨域资源共享CORS介绍

## 2.1 两种验证模式

跨域资源共享CORS的验证机制分两种模式：简单请求和预先请求。

当请求**同时满足下面三个条件时**，CORS验证机制会使用简单模式进行处理。

1.请求方法是下列之一：

- GET
- HEAD
- POST

2.请求头中的Content-Type请求头的值是下列之一：

- application/x-www-form-urlencoded
- multipart/form-data
- text/plain

3.Fetch规范定义了CORS安全头的集合（跨域请求中自定义的头属于安全头的集合）该集合为：

- Accept
- Accept-Language
- Content-Language
- Content-Type （需要注意额外的限制）
- DPR
- Downlink
- Save-Data
- Viewport-Width
- Width

**否则CORS验证机制会使用预先请求模式进行处理。**

## 2.2 简单请求模式

简单请求模式，浏览器直接发送跨域请求，并在请求头中携带Origin的头，表明这是一个跨域的请求。服务器端接到请求后，会根据自己的跨域规则，通过Access-Control-Allow-Origin和Access-Control-Allow-Methods响应头，来返回验证结果。

[![简单请求模式](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200602114829.png)](https://cdn.nlark.com/lark/0/2018/png/18611/1536116421098-6ea68739-6fbe-49b6-8c41-e8a0cb4c1855.png)

应答中携带了跨域头 Access-Control-Allow-Origin。使用 Origin 和 Access-Control-Allow-Origin 就能完成最简单的访问控制。本例中，服务端返回的 Access-Control-Allow-Origin: * 表明，该资源可以被任意外域访问。如果服务端仅允许来自 [http://www.aliyun.com](http://www.aliyun.com/) 的访问，该首部字段的内容如下：

```
Access-Control-Allow-Origin: http://www.aliyun.com
```

现在，除了 [http://www.aliyun.com](http://www.aliyun.com/) ，其它外域均不能访问该资源。

## 2.3 预先请求模式

浏览器在发现页面发出的请求非简单请求，并不会立即执行对应的请求代码，而是会触发预先请求模式。预先请求模式会先发送Preflighted requests（预先验证请求），Preflighted requests是一个OPTION请求，用于询问要被跨域访问的服务器，是否允许当前域名下的页面发送跨域的请求。在得到服务器的跨域授权后才能发送真正的HTTP请求。

OPTIONS请求头部中会包含以下头部：Origin、Access-Control-Request-Method、Access-Control-Request-Headers。服务器收到OPTIONS请求后，设置Access-Control-Allow-Origin、Access-Control-Allow-Method、Access-Control-Allow-Headers、Access-Control-Max-Age头部与浏览器沟通来判断是否允许这个请求。如果Preflighted requests验证通过，浏览器才会发送真正的跨域请求。

[![预先请求模式](D:\PersonalFiles\微云同步助手\图片\Typora图床\1536116498475-1e5df4f7-56ac-482d-add8-77d3b967eb39.png)](https://cdn.nlark.com/lark/0/2018/png/18611/1536116498475-1e5df4f7-56ac-482d-add8-77d3b967eb39.png)

请求中的跨域头 Access-Control-Request-Method 告知服务器，实际请求将使用 GET 方法。请求中的跨域头 Access-Control-Request-Headers 告知服务器，实际请求将携带两个自定义请求首部字段：x-ca-nonce 与 content-type。服务器据此决定该实际请求是否被允许。

应答中的跨域头 Access-Control-Allow-Methods 表明服务器允许客户端使用 GET 方法发起请求。值为逗号分割的列表。

应答中的跨域头 Access-Control-Allow-Headers 表明服务器允许请求中携带字段 x-ca-nonce 与 content-type。与 Access-Control-Allow-Methods 一样，Access-Control-Allow-Headers 的值为逗号分割的列表。

应答中的跨域头 Access-Control-Max-Age 表明该响应的有效时间为 86400 秒，也就是 24 小时。**在有效时间内，浏览器无须为同一请求再次发起预检请求**。请注意，浏览器自身维护了一个最大有效时间，如果该首部字段的值超过了最大有效时间，将不会生效。

# 三.在API网关实现CORS跨域资源共享

## 3.1实现简单请求模式

**API网关默认所有API允许跨域访问，因此如果用户的API后端服务的应答中不做特殊返回，API网关会返回允许所有域跨域访问的相关头**，下面是一个示例：

**客户端的API请求**

```
GET /simple HTTP/1.1Host: www.alibaba.comorgin: http://www.aliyun.comcontent-type: application/x-www-form-urlencoded; charset=utf-8accept: application/json; charset=utf-8date: Mon, 18 Sep 2017 09:53:23 GMT
```

**后端服务应答**

```
HTTP/1.1 200 OKDate: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8Content-Length: 12{"200","OK"}
```

**API网关应答**

```
HTTP/1.1 200 OKDate: Mon, 18 Sep 2017 09:53:23 GMTAccess-Control-Allow-Origin: *X-Ca-Request-Id: 104735BD-8968-458F-9929-DBFA43F324C6Content-Type: application/json; charset=UTF-8Content-Length: 12{"200","OK"}
```

从上面三个报文可以看出，API网关会对用户的后端服务应答做一定修改，增加一个跨域头：

```
Access-Control-Allow-Origin: *
```

这个跨域头的意思是，本API允许所有域的请求访问。

**如果用户需要定制针对简单请求的应答的跨域头，只需要在后端服务应答中，增加Access-Control-Allow-Origin这个跨域头即可，后端服务应答中的头会默认覆盖掉API网关自己增加的头。下面是一个例子，这个例子中的API只允许[http://www.aliyun.com](http://www.aliyun.com/) 这一个域访问：**

**客户端的API请求**

```
GET /simple HTTP/1.1Host: www.alibaba.comorgin: http://www.aliyun.comcontent-type: application/x-www-form-urlencoded; charset=utf-8accept: application/json; charset=utf-8date: Mon, 18 Sep 2017 09:53:23 GMT
```

**后端服务应答**

```
HTTP/1.1 200 OKAccess-Control-Allow-Origin: http://www.aliyun.com Date: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8Content-Length: 12{"200","OK"}
```

**API网关应答**

```
HTTP/1.1 200 OKAccess-Control-Allow-Origin: http://www.aliyun.com X-Ca-Request-Id: 104735BD-8968-458F-9929-DBFA43F324C6Date: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8Content-Length: 12{"200","OK"}
```

## 3.2 实现预先请求模式

API网关允许用户设置方法为OPTIONS的API，并且将后端服务的OPTIONS应答透传给客户端。新建方法为OPTIONS的API，定义的其他部分与正常API一样，有两点需要注意：

- **定义API认证方式时选择无认证；**[![options-api-certificate.png](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200602114745.png)](https://cdn.nlark.com/lark/0/2018/png/18611/1535902222901-272a1900-401b-4398-9785-518e44f0d57b.png)

- **定义API请求时，需要设置path为/，并且匹配所有子路径。选定方法为OPTIONS，API网关控制台会默认设置请求模式为透传模式，且不可修改，用户不需要定义请求参数；**[![方法为OPTIONS的API定义](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200602114750.png)](https://cdn.nlark.com/lark/0/2018/png/18611/1535903251314-6d60e87c-7256-4a4e-bb00-02fc1ad74ce7.png)

用户可以在每个API分组下建立一个方法的OPTIONS的API，来定义这一组API绑定的域名的跨域资源策略。用户可以是用CURL方法来测试自己的跨域API应答情况，下面是针对一个定义好的OPTIONS的API访问的一个示例：

```
sudo curl -X OPTIONS -H "Access-Control-Request-Method:POST" -H "Access-Control-Request-Headers:X-CUSTOM-HEADER" http://ec12ac094e734544be02c928366b7b26-cn-qingdao.alicloudapi.com/optinstest -iHTTP/1.1 200 OKServer: TengineDate: Sun, 02 Sep 2018 15:32:19 GMTConnection: keep-aliveAccess-Control-Allow-Origin: *Access-Control-Allow-Methods: GET,POST,PUT,DELETE,HEAD,OPTIONS,PATCHAccess-Control-Allow-Headers: X-CUSTOM-HEADERAccess-Control-Max-Age: 172800X-Ca-Request-Id: 1016AC86-E345-405C-8049-A6C24078F65F
```

用户在实现方法为OPTIONS的API的时候需要注意的一点是：**API网关会对用户的后端服务应答做一定修改，增加四个跨域头（Access-Control-Allow-Origin、Access-Control-Allow-Methods、Access-Control-Allow-Headers、Access-Control-Max-Age），后端服务应答中，需要返回所有跨域头来覆盖API网关默认跨域头。**

下面是一个完整的预先请求模式的请求与应答示例。

**客户端的方法为OPTIONS的API请求**

```
OPTIONS /simple HTTP/1.1Host: www.alibaba.comorgin: http://www.aliyun.comAccess-Control-Request-Method: POSTAccess-Control-Request-Headers: X-PINGOTHER, Content-Typeaccept: application/json; charset=utf-8date: Mon, 18 Sep 2017 09:53:23 GMT
```

**后端服务应答**

```
HTTP/1.1 200 OKAccess-Control-Allow-Origin: http://www.aliyun.com Access-Control-Allow-Methods: GET,POSTAccess-Control-Allow-Headers: X-CUSTOM-HEADERAccess-Control-Max-Age: 10000Date: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8
```

**API网关应答**

```
HTTP/1.1 200 OKAccess-Control-Allow-Origin: http://www.aliyun.com Access-Control-Allow-Methods: GET,POSTAccess-Control-Allow-Headers: X-CUSTOM-HEADERAccess-Control-Max-Age: 10000X-Ca-Request-Id: 104735BD-8968-458F-9929-DBFA43F324C6Date: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8
```

**客户端发送正常业务请求**

```
GET /simple HTTP/1.1Host: www.alibaba.comorgin: http://www.aliyun.comcontent-type: application/x-www-form-urlencoded; charset=utf-8accept: application/json; charset=utf-8date: Mon, 18 Sep 2017 09:53:23 GMT
```

**后端服务应答**

```
HTTP/1.1 200 OKDate: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8Content-Length: 12{"200","OK"}
```

**API网关应答**

```
HTTP/1.1 200 OKAccess-Control-Allow-Origin: *Access-Control-Allow-Methods: GET,POST,PUT,DELETE,HEAD,OPTIONS,PATCHAccess-Control-Allow-Headers: X-Requested-With,X-Sequence,X-Ca-Key,X-Ca-Secret,X-Ca-Version,X-Ca-Timestamp,X-Ca-Nonce,X-Ca-API-Key,X-Ca-Stage,X-Ca-Client-DeviceId,X-Ca-Client-AppId,X-Ca-Signature,X-Ca-Signature-Headers,X-Forwarded-For,X-Ca-Date,X-Ca-Request-Mode,Authorization,Content-Type,Accept,Accept-Ranges,Cache-Control,Range,Content-MD5Access-Control-Max-Age: 172800X-Ca-Request-Id: 104735BD-8968-458F-9929-DBFA43F324C6Date: Mon, 18 Sep 2017 09:53:23 GMTContent-Type: application/json; charset=UTF-8Content-Length: 12{"200","OK"}
```