# 记录一次从Java11降为Java8的过程

## HttpClient

java.net.http包是JDK11的功能，降级到JDK8之后，无法继续使用，采用ApacheHttpClient作为代替，创建了一个工具类

```java
package com.shenlanbao.consult.common.utils;

import com.aliyun.openservices.shade.org.apache.commons.lang3.StringUtils;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpResponse;
import org.apache.http.HttpStatus;
import org.apache.http.NameValuePair;
import org.apache.http.client.HttpClient;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.net.URI;
import java.nio.charset.StandardCharsets;
import java.util.*;

/**
 * Apache HttpClient工具类，
 * 背景：用于处理Java版本降级后(11 -> 8)，替换原有java.net.http包，
 * 调用第三方服务，发送Http请求
 * @author Rot
 * @date 2020/5/25 17:10
 */
@Slf4j
public class HttpClientUtils {

    /**
     * 处理get请求
     * @param url 请求路径
     * @param params 参数（Map格式）
     * @return 请求结果
     */
    public static String doGet(String url, Map<String, String> params) {

        // 返回结果
        String result = null;
        // 创建HttpClient对象
        HttpClient httpClient = HttpClientBuilder.create().build();
        HttpGet httpGet = null;
        try {
            // 拼接参数,可以用URIBuilder,也可以直接拼接在?传值，拼在url后面，如下--httpGet = new
            // HttpGet(uri+"?id=123");
            URIBuilder uriBuilder = new URIBuilder(url);
            if (null != params && !params.isEmpty()) {
                for (Map.Entry<String, String> entry : params.entrySet()) {
                    // 注意：setParameter会覆盖同名参数的值，addParameter则不会
                    uriBuilder.addParameter(entry.getKey(), entry.getValue());
                }
            }
            URI uri = uriBuilder.build();
            // 创建get请求
            httpGet = new HttpGet(uri);
            log.info("发送Get请求：" + uri);
            HttpResponse response = httpClient.execute(httpGet);
            // 当状态码为200的时候，返回结果集
            result = EntityUtils.toString(response.getEntity());
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                // 结果返回
                log.info("请求成功,url={},返回数据={}", uri, result);
            } else {
                log.info("请求失败,url={},返回数据={}", uri, result);
            }
        } catch (Exception e) {
            log.error("请求异常,url={},msg={}", url, e.getMessage());
            e.printStackTrace();
        } finally {
            // 释放连接
            if (null != httpGet) {
                httpGet.releaseConnection();
            }
        }
        return result;
    }

    /**
     * 处理post请求（请求体为表单）
     * @param url 请求路径
     * @param params 请求参数(Map)
     * @return 请求结果
     */
    public static String doPost(String url, Map<String, String> params) {
        String result = null;
        // 创建httpclient对象
        HttpClient httpClient = HttpClientBuilder.create().build();
        HttpPost httpPost = null;
        try {
            httpPost = new HttpPost(url);
            // 参数键值对
            if (null != params && !params.isEmpty()) {
                List<NameValuePair> pairs = new ArrayList<NameValuePair>();
                NameValuePair pair = null;
                for (String key : params.keySet()) {
                    pair = new BasicNameValuePair(key, params.get(key));
                    pairs.add(pair);
                }
                // 模拟表单
                UrlEncodedFormEntity entity = new UrlEncodedFormEntity(pairs);
                httpPost.setEntity(entity);
            }
            HttpResponse response = httpClient.execute(httpPost);
            result = EntityUtils.toString(response.getEntity(), "utf-8");
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                log.info("请求成功,url={},返回数据={}", url, result);
            } else {
                log.info("请求失败,url={},返回数据={}", url, result);
            }
        } catch (Exception e) {
            log.error("请求异常,url={},msg={}", url, e.getMessage());
            e.printStackTrace();
        } finally {
            if (null != httpPost) {
                // 释放连接
                httpPost.releaseConnection();
            }
        }
        return result;
    }

    /**
     * 处理post请求（请求体为json字符串）
     * @param url
     * @param params
     * @return
     */
    public static String sendJsonStr(String url, String params) {
        String result = null;
        log.info("发送Json请求体,url={},body={}", url, params);

        HttpClient httpClient = HttpClientBuilder.create().build();
        HttpPost httpPost = new HttpPost(url);
        try {
            httpPost.addHeader("Content-type", "application/json; charset=utf-8");
            httpPost.setHeader("Accept", "application/json");
            if (!StringUtils.isEmpty(params)) {
                httpPost.setEntity(new StringEntity(params, StandardCharsets.UTF_8));
            }
            HttpResponse response = httpClient.execute(httpPost);
            result = EntityUtils.toString(response.getEntity());
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                log.info("请求成功,url={},返回数据={}", url, result);
            } else {
                log.info("请求失败,url={},返回数据={}", url, result);
            }
        } catch (IOException e) {
            log.error("请求异常,url={},msg={}", url, e.getMessage());
            e.printStackTrace();
        }
        return result;
    }
}

```

## Duration.toSeconds()

源码：

```java
Duration between = Duration.between(smsSendLog.getCreatedAt(), LocalDateTime.now());
return between.toSeconds() < sendInSeconds;
```

降级后报错，修改为：

```java
Duration between = Duration.between(smsSendLog.getCreatedAt(), LocalDateTime.now());
return between.getSeconds() < sendInSeconds;
```

说明：

在Java8中，有如下几个转换时间单位的方法：

```java
toNanos();
toMillis();
toMinutes();
toHours();
toDays();
```

但是唯独没有toSeconds()方法（其实有，但是是个私有方法），这是因为已经有getSeconds()这个方法能达到同样的功能了。

推测之后的版本中为了统一，增加了toSeconds方法，实际结果是一样的，测试用例：

```java
public static void main(String[] args) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    LocalDateTime oldTime = LocalDateTime.parse("2020-01-01 00:00:00", formatter);
    LocalDateTime newTime = LocalDateTime.parse("2020-01-01 01:00:00", formatter);
    Duration duration = Duration.between(oldTime, newTime);
    System.out.println("getSeconds:" + duration.getSeconds());
    System.out.println("toSeconds:" + duration.toSeconds());
}

/*
执行结果：
	getSeconds:3600
	toSeconds:3600
*/
```

## Set.of()、List.of()、Map.of()

Jdk9新增了Set.of()、List.of()、Map.of()这三个API，可以快速的生成对应的集合

从Jdk11降级为Jdk8后，上面的三个方法报错，

源码：

```java
this.beneficiaryRelationToInsurant = Map.of("1", "本人", "2", "父母", "3", "子女", "4", "配偶", "8", "其他").get(type);
```

解决方法：

手动创建Map集合，并插入数据

```java
this.beneficiaryRelationToInsurant = new HashMap<String, String>(){{
        put("1", "本人");
        put("2", "父母");
        put("3", "子女");
        put("4", "配偶");
        put("8", "其他");
}}.get(type);
```

## Optional.isEmpty()

Java8中Optional类没有加入isEmpty()的方法，与其等效的是`!Optional.isPresent()`

## String.strip()

Jdk11对String引入了strip()这个API，用来去除字符串之前和之后的半角或者全角空白字符。

在此之前的trim()方法只能去除半角空白字符。

解决方法

一、替换掉所有全角和半角的空格，缺点是字符串中的空格也会被替换

```java
str.replaceAll("　| ", "");
```

二、写一个方法来处理，循环判断然后截取字符串                                     

```java
public static String strip(String a) {  
    a = a.trim();  
    while(a.startsWith("　")) {  
       a = a.substring(1,a.length()).trim();  
    }  
    while(a.endsWith("　")) {  
       a = a.substring(0,a.length()-1).trim();  
    }
    return a; 
}  
```

