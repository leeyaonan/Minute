# 2    Guns整合JWT签名机制 

## 2.1  介绍

### 2.1.1 基本流程

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212521.jpg)

 

## 2.2  实践：

### 2.2.1 第一步，开启配置

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212532.png)

### 2.2.2 第二步 去登录获取钥匙

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212544.png)


请求这个路径，传入用户名和密码，获取钥匙

![](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212554.png)

响应报文：

```json
{
"randomKey": "yvg1ye",
"token":"eyJhbGciOiJIUzUxMiJ9.eyJyYW5kb21LZXkiOiJ5dmcxeWUiLCJzdWIiOiJhZG1pbiIsImV4cCI6MTU1NTg0ODk0MiwiaWF0IjoxNTU1MjQ0MTQyfQ.clEVOLBNQr4dFAeK_4fEpPlzCo-zDjNxEsPxyBftbmjJRLGY9rrdSHUTmeckk-v8XCKmPKlxqXICyWkKGYZ7jA"
}
```

这个token就是我们的钥匙

### 2.2.3 使用token

#### 2.2.3.1 选择1：对数据不签名如果是不带参数 或者没有在请求正文

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212727.jpg)

 

#### 2.2.3.2 选择2： 需要对数据进行签名

第三步 客户端（前端），对即将传入服务器的数据进行签名

上面的盐值需要使用，token带回来的randomkey

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212800.jpg)

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212808.jpg)

把提交的数据，

进行base64加密（编码）操作，得到encode 也就是object值

然后把encode和随机盐值 进行md5计算签名。得到如下：

```json
{
"object":"eyJjaWQiOjIyMiwiZGVzY3JpcHRpb24iOiJhYWFhIiwiZXN0b3JlUHJpY2UiOjIyLjAsImltZ1VybCI6Ii9hZmRhc2QvYWRzZnNhZGYiLCJtYXJrUHJpY2UiOjQ0LjAsInBpZCI6MTgsInBuYW1lIjoi6LW15peg5p6BIiwicG51bSI6MjIyfQ==",
"sign":"cddb228fbfac92415afa12ae57861fd1"
}
```

这就是经过签名之后的钥匙形式。

##### 2.2.3.2.1     第四步 将签名以后的数据传到服务器

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212849.jpg)

 

##### 2.2.3.2.2     第五步，服务器拿到数据并验证

拦截器里验证token的有效性。

Token有效即放行。

请求参数封装之前验证数据签名的有效性。签名ok做请求参数封装。

结果如下：

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212908.jpg)

 

说明使用JWT token验证成功。数据成功插入数据库！

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200527212919.jpg)


