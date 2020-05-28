## 1. OpenID Connect简介

OpenID Connect是2014年初发布的开放标准，定义了一种基于OAuth2的可互操作的方式来来提供用户身份认证。它使用简单的REST/JSON消息流来实现，和之前任何一种身份认证协议相比，开发者可以轻松集成。OpenID Connect使用 API 进行身份交互的框架，允许客户端根据授权服务器的认证结果最终确认用户的身份，以及获取基本的用户信息；支持包括Web、移动、JavaScript在内的所有客户端类型；它是可扩展的协议，允许你使用某些可选功能，如身份数据加密、OpenID提供商发现、会话管理等。

### 1.1 OpenID Connect 角色介绍

1. 客户端：直接为终端用户提供服务
2. 认证服务：OpenID 提供方，通常是一个 OpenID 认证服务器，它能为第三方颁发用于认证的ID Token
3. 业务服务：提供业务服务，比如查询用户资料
4. 终端用户：指持有资源拥有人

### 1.2 OpenID Connect 流程描述

[![OpenID Connect流程描述](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200528141856.png)](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555992896926/DCCF0C9A-05DD-45CD-A3D9-953CFAE1780C.png)

1. 客户端发送认证请求给认证服务；
2. 终端用户在认证页面进行授权确认（可选）；
3. 认证服务对认证请求进行验证，发送ID Token给客户端；
4. 客户端向业务请求，请求中携带ID Token；
5. 业务服务验证ID Token是否合法后返回业务应答；

这个流程的第二步是可选的，如果终端用户在客户端输入了用户名和密码，第一步中的认证请求中携带了用户名和密码，那么认证服务在验证了用户名和密码后，省去第二步，可以直接在应答中返ID Token。这种模式更加简洁，阿里云的API网关的OpenID Connect验证模式就是这种，本文将在第二节仔细描述。

### 1.3 JWT(JSON Web Token)格式数据

认证服务返回的ID Token需要严格遵守JWT(JSON Web Token)的定义，下面是JWT(JSON Web Token)的定义细节：

> 1. iss：必须。Issuer Identifier，认证服务的唯一标识，一个区分大小写的https URL，不包含query和fragment组件
> 2. sub：必须。Subject Identifier，iss提供的终端用户的标识，在iss范围内唯一，最长为255个ASCII个字符，区分大小写
> 3. aud：必须。Audience(s)，标识ID Token的受众，必须包含OAuth2的client_id，分大小写的字符串数组
> 4. exp：必须。Expiration time，超过此时间的ID Token会作废
> 5. iat：必须。Issued At Time，JWT的构建的时间
> 6. auth_time：AuthenticationTime，终端用户完成认证的时间。
> 7. nonce：发送认证请求的时候提供的随机字符串，用来减缓重放攻击，也可以用来关联客户端Session。如果nonce存在，客户端必须验证nonce
> 8. acr：可选。Authentication Context Class Reference，表示一个认证上下文引用值，可以用来标识认证上下文类
> 9. amr：可选。Authentication Methods References，表示一组认证方法
> 10. azp：可选。Authorized party，结合aud使用。只有在被认证的一方和受众（aud）不一致时才使用此值，一般情况下很少使用

下面是一个典型ID Token的示例，供参考

```json
{
    "iss": "https://1.2.3.4:8443/auth/realms/kubernetes",
    "sub": "547cea22-fc8a-4315-bdf2-6c92592a6e7c",
    "aud": "kubernetes",
    "exp": 1525158711,
    "iat": 1525158411,
    "auth_time": 0,
    "nonce": "n-0S6_WzA2Mj",
    "acr": "1",
    "azp": "kubernetes",
    "nbf": 0,
    "typ": "ID",
    "session_state": "150df80e-92a1-4b0c-a5c5-8c858eb5a848",
    "userId": "123456",
    "preferred_username": "theone",
    "given_name": "the",
    "family_name": "one",
    "email": "theone@mycorp.com"
}
```

关于ID Token的更详细的定义请参考： [https://openid.net/specs/openid-connect-core-1_0.html](https://openid.net/specs/openid-connect-core-1_0.html?spm=a2c4g.11186623.2.16.40647f96fqwejU)