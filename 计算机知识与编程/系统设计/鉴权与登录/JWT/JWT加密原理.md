# [JWT加密原理](https://www.cnblogs.com/buou/articles/11261252.html)

## JWT加密原理

### JWT：

JSON Web Token的缩写，是REST接口的一种安全策略，也是一种安全的规范，使用JWT可以让我们在用户端和服务端建立一种可靠的通信保障。

 

### 使用简介：

用户端登录成功后，服务器会生成一个token给到用户端，token里面包含用户信息（用户id），用户后续携带该token进行请求访问，服务器从用户的请求中成功解析token信息，则说明用户的请求合法有效，反之服务器从token解析不出用户信息就认为请求非法失效。

 

### 优点：

在分布式系统中，可以有效的解决单点登录问题以及SESSION共享问题；服务器不保存token或者用户session信息，可以减少服务器压力。

### 缺点：

没有失效策略；设置失效时间后，只能等待token过期，无法改变token里面的失效时间

 

### 组成：

一个JWT是有三个部门组成：头部（header），消息体（playload），签名（sign）。

 

### 详细说明：

#### 头部（header）:

```json
{ "typ": "JWT", "alg": "HS256"}　　
```

typ是type的缩写，说明类型是JWT，alg是加密加密方式为HS256。上面的内容用BASE64加密后的内容如下：

```json
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

####  

#### 消息体（playload）：

该部分是实际存放信息的地方，JSON格式如下：

```json
{ "iss": "why", "iat": 1416797419, "exp": 1448333419, "aud": "www.example.com", ``"sub": "taobao.com"}
```

iss: 该JWT的签发者，是否使用是可选的；
sub: 该JWT所面向的用户，是否使用是可选的；
aud: 接收该JWT的一方，是否使用是可选的；
exp(expires): 什么时候过期，这里是一个Unix时间戳，是否使用是可选的；
iat(issued at): 在什么时候签发的(UNIX时间)，是否使用是可选的；
nbf (Not Before)：如果当前时间在nbf里的时间之前，则Token不被接受；一般都会留一些余地，比如几分钟；，是否使用是可选的。
同样，上面的内容用BASE64加密后如下；

```
ewogICJpc3MiOiAid2h5IiwgCiAgImlhdCI6IDE0MTY3OTc0MTksIAogICJleHAiOiAxNDQ4MzMzNDE5LCAKICAiYXVkIjogInd3dy5leGFtcGxlLmNvbSIsIAogICJzdWIiOiAidGFvYmFvLmNvbSIsIAp9
```

####  

#### 签名（sign）：

签名是对头部和消息体的内容进行签名，即便有人获取token的内容，改变消息题或者头部的内容，那么生成的签名将和现在的签名不一样；同时如果不知道服务器加密时用的密钥的话，得出来的签名也一定是不一样的。

##### 签名的过程：

采用header中声明的算法，将base64加密后的header、base64加密后的playload以及密钥（secret）进行计算得到。

##### 如以上要签名的内容为：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.ewogICJpc3MiOiAid2h5IiwgCiAgImlhdCI6IDE0MTY3OTc0MTksIAogICJleHAiOiAxNDQ4MzMzNDE5LCAKICAiYXVkIjogInd3dy5leGFtcGxlLmNvbSIsIAogICJzdWIiOiAidGFvYmFvLmNvbSIsIAp9
```

采用header声明的HS256算法进行加密，同时提供一个密钥（secret），计算签名后的内容：

```
6OcLdX38eKWn1gCyJ6RNwsAvIvXxYq1CGBkFWgiTsyc
```

##### 最后将三者拼接在一起就是JWT：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.ewogICJpc3MiOiAid2h5IiwgCiAgImlhdCI6IDE0MTY3OTc0MTksIAogICJleHAiOiAxNDQ4MzMzNDE5LCAKICAiYXVkIjogInd3dy5leGFtcGxlLmNvbSIsIAogICJzdWIiOiAidGFvYmFvLmNvbSIsIAp9.6OcLdX38eKWn1gCyJ6RNwsAvIvXxYq1CGBkFWgiTsyc
```

 

### 解密：

后端服务校验jwtToken是否有权访问接口服务，进行解密认证，如校验访问者的userid，首先 
用将字符串按.号切分三段字符串，分别得到header和playload和sign。然后将header.playload拼装用密钥和HAMC SHA-256算法进行加密然后得到新的字符串和sign进行比对，如果一样就代表数据没有被篡改，然后从头部取出exp对存活期进行判断，如果超过了存活期就返回空字符串，如果在存活期内返回userid的值。

 

### 安全性：

JWT的header和playload都是简单的使用base64加密的， 可以解密获取里面的内容，所以通过http传输不安全。

1.使用https进行ssl加密传输，保证通道安全；

2.playload中尽量不要放置敏感信息，只保存用户唯一标实即可。

 

### 参考：

https://blog.csdn.net/why15732625998/article/details/78534711

https://www.cnblogs.com/royalluren/p/11041888.html

 