# [API网关 OpenID Connect 使用指南](https://help.aliyun.com/document_detail/48019.html?spm=5176.14097614.0.0.6d855e1aZX50G2)

更新时间：2020-02-10 14:08:54

  

[本页目录](javascript:void(0))

- [1. OpenID Connect简介](https://help.aliyun.com/document_detail/48019.html?spm=5176.14097614.0.0.6d855e1aZX50G2#h2-1-openid-connect-1)
- [2. API网关OpenID Connect业务流程](https://help.aliyun.com/document_detail/48019.html?spm=5176.14097614.0.0.6d855e1aZX50G2#h2-2-api-openid-connect-2)
- [3. 在API配置OpenID Connect](https://help.aliyun.com/document_detail/48019.html?spm=5176.14097614.0.0.6d855e1aZX50G2#h2-3-api-openid-connect3)
- [4. 后端服务实现ID Token颁发](https://help.aliyun.com/document_detail/48019.html?spm=5176.14097614.0.0.6d855e1aZX50G2#h2-4-id-token-4)
- [5. 常见问题](https://help.aliyun.com/document_detail/48019.html?spm=5176.14097614.0.0.6d855e1aZX50G2#h2-5-5)

阿里云API网关在OpenID Connect协议的基础上实现了一套基于用户体系对用户的API进行授权访问的机制，满足用户个性化安全设置的需求。

## 1. OpenID Connect简介

OpenID Connect是2014年初发布的开放标准，定义了一种基于OAuth2的可互操作的方式来来提供用户身份认证。它使用简单的REST/JSON消息流来实现，和之前任何一种身份认证协议相比，开发者可以轻松集成。OpenID Connect使用 API 进行身份交互的框架，允许客户端根据授权服务器的认证结果最终确认用户的身份，以及获取基本的用户信息；支持包括Web、移动、JavaScript在内的所有客户端类型；它是可扩展的协议，允许你使用某些可选功能，如身份数据加密、OpenID提供商发现、会话管理等。

### 1.1 OpenID Connect 角色介绍

1. 客户端：直接为终端用户提供服务
2. 认证服务：OpenID 提供方，通常是一个 OpenID 认证服务器，它能为第三方颁发用于认证的ID Token
3. 业务服务：提供业务服务，比如查询用户资料
4. 终端用户：指持有资源拥有人

### 1.2 OpenID Connect 流程描述

[![OpenID Connect流程描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200528141856.png)](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555992896926/DCCF0C9A-05DD-45CD-A3D9-953CFAE1780C.png)

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

## 2. API网关OpenID Connect业务流程

阿里云API网关将OpenID Connect作为用户自主授权API的一种认证模式。作为处于客户端和后端服务中间的API网关，可以为后端服务去验证ID Token的合法性，并将没有拿到后端服务颁发的合法ID Token的请求在API网关层面给拒绝了。

### 2.1 准备条件

用户需要在API网关上配置以下两点，才能正常使用API网关的OpenID Connect认证模式：

1. 配置一个获取授权API，并且在这个API上指定一个公钥；
2. 所有需要保护的业务API设置成为OpenID Connect的业务API；

具体的配置方法参见第三章。

### 2.2 业务流程

[![API网关OpenID Connect业务流程](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200528141937.png)](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555992984147/9FA7DBC2-C17D-416F-AC1A-04E1F60800FA.png)

上图是API网关处理OpenID Connect的的整个业务流程，下面我们用文字来详细描述下：

1. 客户端向API网关发起认证请求，请求中一般会携带终端用户的用户名和密码；
2. API网关将请求直接转发给后端服务；
3. 后端服务读取请求中的验证信息（比如用户名、密码）进行验证，验证通过后使用私钥生成标准的ID Token，返回给API网关；
4. API网关将携带ID Token的应答返回给客户端，客户端需要将这个ID Token缓存到本地；
5. 客户端向API网关发送业务请求，请求中携带ID Token；
6. API网关使用用户设定的公钥对请求中的ID Token进行验证，验证通过后，将请求透传给后端服务；
7. 后端服务进行业务处理后应答；
8. API网关将业务应答返回给客户端。

在这个整个过程中，API网关利用OpenID Connect的认证机制，实现了用户使用自己的用户体系对自己API进行授权的能力。

### 2.3 授权范围与时效

API网关会认为用户颁发的ID Token有权利访问整个分组下的所有OpenID Connect的业务API。如果需要更细力度的权限管理，还需要后端服务自己解开ID Token进行权限认证。API网关会验证ID Token中的exp字段，一旦这个字段过期了，API网关会认为这个ID Token无效而将请求直接打回。过期时间这个值必须设置，并且过期时间一定要小于7天。

## 3. 在API配置OpenID Connect

本章主要教大家如何在API网关配置OpenID Connect，主要有三步：

1. 生成一个公私秘钥对；
2. 配置一个授权API，在这个API上将公钥配置好；
3. 所有业务API安全认证类型设置为OpenID Connect模式。下面是具体细节。

### 3.1 准备公钥与私钥

用户可以在 这个站点[https://mkjwk.org](https://mkjwk.org/?spm=a2c4g.11186623.2.18.40647f96fqwejU) 生成用于ID Token生成与验证的公钥与私钥，目前API网关支持的秘钥对的加密算法为RSA SHA256，秘钥对的加密的位数为2048。

[![准备公钥与私钥](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200528141943.png)](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555993015595/7D654BB1-0329-4D78-A71E-016FC4595A5E.png)

在这个网站生成的公钥用于本文3.1中设置授权API：

```json
{
    "kty": "RSA",
    "e": "AQAB",
    "kid": "uniq_key",
    "alg": "RS256",
    "n": "gbnCVY4XxM-MB1mseAJnIItognv3LRuHkVv5W-gF-yvXnYfi8t-L33oF73i4eyE9i4uSElP-CCoSbyJUJhhNSS7_njDp6Ex3WNq0KimRSmXanD5453CFBrgxJlai4aaZxPYsIdjqiVijAris40gRVZQ7aMtDc6Rmu1IS3564dYJ0YR9GA8tZPvuX8ESHnSXgQLM8y6BBLpoojSKajApOK1QW6RieQZZcMIuMjOoJb9NWHqSDFm7PXYeQWxpH_HfN1wMPo7tln5vWbi-vFIHJanWF1syZ9XR0yba53YmMBj7-YDuxF3_sTK-9I8upWGEC9M16Qn-C7eHcDMLZ8XoSUw"
}
```

在这个网站生成的秘钥对用于本文4章中生成IdToken编程时使用：

```json
{
    "kty": "RSA",
    "d": "HSmya2NXKpJx61EYeZ4oquNMKlFN_uD6eA4SH7woZA-2GB7tQSZKHoIjBXPBHUUavd0xiFdDe3hhzoQMIMhDz5j2NAzQ-Lz_84SvDe9sTypYm9lbesQL07firLi7Qzkdxm6E-1L1Xs0DUGBN1YZlBzUcqfFQB5ZE1gWcYpMe6qOHkJWkb5GaiZm_x3D-fUYO5VV7t0G-NtrF5FAs06qVW7fgMqeOi6l9_-Nyldc1XKAOrmqzOu5GqgLMkyN76I_FYikNQiTdvReR5lg6YYULH0rPcKsDBllalAOz1HZQhYK7xC9AyYN2iEyQwlQWwepirs1taUTsH6YoegeK2sazwQ",
    "e": "AQAB",
    "kid": "uniq_key",
    "alg": "RS256",
    "n": "gbnCVY4XxM-MB1mseAJnIItognv3LRuHkVv5W-gF-yvXnYfi8t-L33oF73i4eyE9i4uSElP-CCoSbyJUJhhNSS7_njDp6Ex3WNq0KimRSmXanD5453CFBrgxJlai4aaZxPYsIdjqiVijAris40gRVZQ7aMtDc6Rmu1IS3564dYJ0YR9GA8tZPvuX8ESHnSXgQLM8y6BBLpoojSKajApOK1QW6RieQZZcMIuMjOoJb9NWHqSDFm7PXYeQWxpH_HfN1wMPo7tln5vWbi-vFIHJanWF1syZ9XR0yba53YmMBj7-YDuxF3_sTK-9I8upWGEC9M16Qn-C7eHcDMLZ8XoSUw"
}
```

### 3.2 配置授权API和业务API

#### 3.2.1 授权API

下图是配置授权API的第一页，也是最重要的一页，除了这一页以外，授权API的其他配置均和普通的API配置相同。

[![授权API设置](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200528142154.jpg)](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555993077510/18_37_14__04_16_2019.jpg)

这一页中标注出来的注意项如下：

1. 安全认证：选择OpenID Connect或者 OpenID Connect & 阿里云APP，这两者的区别是，后者除了验证OpenID Connect的ID Token之外，还需要验证阿里云自己的APP，签名，具体签名算法参见 [https://help.aliyun.com/document_detail/29475.html](https://help.aliyun.com/document_detail/29475.html?spm=a2c4g.11186623.2.21.40647f96fqwejU)
2. OpenID Connect 模式：选择获取授权API；
3. KeyId，填写一个分组范围内唯一的秘钥名称，尽量使用英文加数字；
4. 公钥，填写刚才生成的公钥，是整个JSON串都需要填入。

#### 3.2.2 业务API

下图是配置OpenID Connect 业务API的第一页，也是最重要的一页，除了这一页以外，OpenID Connect 业务API的其他配置均和普通的API配置相同。[![业务API设置](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555993055048/17_41_27__04_16_2019的副本.jpg)](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/48019/cn_zh/1555993055048/17_41_27__04_16_2019的副本.jpg)

1. 安全认证：选择OpenID Connect或者 OpenID Connect && 阿里云APP，这两者的区别是，后者除了验证OpenID Connect的ID Token之外，还需要验证阿里云自己的APP，签名，具体签名算法参见 [https://help.aliyun.com/document_detail/29475.html](https://help.aliyun.com/document_detail/29475.html?spm=a2c4g.11186623.2.23.40647f96fqwejU)
2. OpenID Connect 模式：选择业务API；
3. Token对应的参数名称，填写请求中对应的ID Token的参数名称，比如idToken；

#### 3.3 通过API插件配置

API网关还可以通过插件方式配置OpenID Connect，具体请参考文档： [https://help.aliyun.com/document_detail/103228.html](https://help.aliyun.com/document_detail/103228.html?spm=a2c4g.11186623.2.24.40647f96fqwejU)

## 4. 后端服务实现ID Token颁发

```java
import java.security.PrivateKey; 
import org.jose4j.json.JsonUtil;
import org.jose4j.jwk.RsaJsonWebKey;
import org.jose4j.jwk.RsaJwkGenerator;
import org.jose4j.jws.AlgorithmIdentifiers;
import org.jose4j.jws.JsonWebSignature;
import org.jose4j.jwt.JwtClaims;
import org.jose4j.jwt.NumericDate;
import org.jose4j.lang.JoseException;
public class GenerateJwtDemo {
    public static void main(String[] args) throws JoseException  {
          //使用在API网关设置的keyId
        String keyId = "uniq_key";
          //使用本文3.2节生成的Keypare
        String privateKeyJson = "{\n"
            + "  \"kty\": \"RSA\",\n"
            + "  \"d\": "
            +
            "\"O9MJSOgcjjiVMNJ4jmBAh0mRHF_TlaVva70Imghtlgwxl8BLfcf1S8ueN1PD7xV6Cnq8YenSKsfiNOhC6yZ_fjW1syn5raWfj68eR7cjHWjLOvKjwVY33GBPNOvspNhVAFzeqfWneRTBbga53Agb6jjN0SUcZdJgnelzz5JNdOGaLzhacjH6YPJKpbuzCQYPkWtoZHDqWTzCSb4mJ3n0NRTsWy7Pm8LwG_Fd3pACl7JIY38IanPQDLoighFfo-Lriv5z3IdlhwbPnx0tk9sBwQBTRdZ8JkqqYkxUiB06phwr7mAnKEpQJ6HvhZBQ1cCnYZ_nIlrX9-I7qomrlE1UoQ\",\n"
            + "  \"e\": \"AQAB\",\n"
            + "  \"kid\": \"myJwtKey\",\n"
            + "  \"alg\": \"RS256\",\n"
            + "  \"n\": \"vCuB8MgwPZfziMSytEbBoOEwxsG7XI3MaVMoocziP4SjzU4IuWuE_DodbOHQwb_thUru57_Efe"
            +
            "--sfATHEa0Odv5ny3QbByqsvjyeHk6ZE4mSAV9BsHYa6GWAgEZtnDceeeDc0y76utXK2XHhC1Pysi2KG8KAzqDa099Yh7s31AyoueoMnrYTmWfEyDsQL_OAIiwgXakkS5U8QyXmWicCwXntDzkIMh8MjfPskesyli0XQD1AmCXVV3h2Opm1Amx0ggSOOiINUR5YRD6mKo49_cN-nrJWjtwSouqDdxHYP-4c7epuTcdS6kQHiQERBd1ejdpAxV4c0t0FHF7MOy9kw\"\n"
            + "}";
        JwtClaims claims = new JwtClaims();
        claims.setGeneratedJwtId();
        claims.setIssuedAtToNow();
        //过期时间一定要设置，并且小于7天
        NumericDate date = NumericDate.now();
        date.addSeconds(120*60);
        claims.setExpirationTime(date);
        claims.setNotBeforeMinutesInThePast(1);
        claims.setSubject("YOUR_SUBJECT");
        claims.setAudience("YOUR_AUDIENCE");
        //添加自定义参数，所有值请都使用String类型
        claims.setClaim("userId", "1213234");
        claims.setClaim("email", "userEmail@youapp.com");
        JsonWebSignature jws = new JsonWebSignature();
        jws.setAlgorithmHeaderValue(AlgorithmIdentifiers.RSA_USING_SHA256);
          //必须设置
        jws.setKeyIdHeaderValue(keyId);
        jws.setPayload(claims.toJson());
        PrivateKey privateKey = new RsaJsonWebKey(JsonUtil.parseJson(privateKeyJson)).getPrivateKey();
        jws.setKey(privateKey);
        String jwtResult = jws.getCompactSerialization();
        System.out.println("Generate Json Web token , result is " + jwtResult);
    }
}
```

以上是生成ID Token的Java示例代码，其中有以下几个地方需要重点关注：

1. keyId需要三个环节都一致，且全局唯一：
   - 在本文3.1节，[https://mkjwk.org](https://mkjwk.org/?spm=a2c4g.11186623.2.25.40647f96fqwejU) 生成秘钥时填写的keyId;
   - 在本文3.2.1节设置的keyId；
   - 代码中的keyId
2. JsonWebSignature 对象的KeyIdHeaderValue必须设置；
3. privateKeyJson 使用3.1节中生成的Keypair Json字符串（三个方框内的第一个）；
4. 过期时间一定要设置，并且小于7天；
5. 添加自定义参数，所有值请都使用String类型；

## 5. 常见问题

### 5.1 APP签名认证

如果安全认证选择“OpenID Connect & 阿里云APP”时，请求中必须携带使用阿里云AK加密的签名，具体签名方式请参见文档： [https://help.aliyun.com/document_detail/29475.html](https://help.aliyun.com/document_detail/29475.html?spm=a2c4g.11186623.2.26.40647f96fqwejU)

### 5.2 Swagger导入支持OPENID CONNECT吗？

目前暂时不支持；

### 5.3 API网关错误应答列表

| StatusCode | ErrorMessage                                    | 含义                                        |
| :--------- | :---------------------------------------------- | :------------------------------------------ |
| 401        | “OpenId Connect Verify Fail, IdToken Not Exist” | 业务APP请求中没有携带定义好的IDToken参数    |
| 401        | “234, JWS set idToken exception”                | 解析IDToken时出现Exception                  |
| 401        | “234, Not found by keyId”                       | 根据Token中的KeyId，找不到用户设置的公钥    |
| 401        | “235, JWS set Public-Key exception”             | 根据Token中的KeyId，找到的公钥不可用        |
| 401        | ”237, Verify signature failed“                  | 验证签名失败                                |
| 401        | “245, IdToken is out of scope”                  | 请求中的IDToken不属于当前分组，无权访问     |
| 401        | “238, JWS get payload exception”                | 请求中的IDToken找不到payload                |
| 401        | “239, Parse payload to JwtClaims exception”     | 转化payload为 JwtClaims 时出错              |
| 401        | “239, idToken expired”                          | 请求中的IDToken已经过期                     |
| 401        | “Invalid OpenId Connect Config”                 | API配置错误，获取不到OpenId Connect相关配置 |