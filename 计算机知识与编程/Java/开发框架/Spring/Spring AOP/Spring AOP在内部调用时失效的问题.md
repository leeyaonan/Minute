# Spring AOP在内部调用时失效的问题

> 背景：在开发过程中，遇到了在同一个类中，使用this调用被AOP强化的方法时，切面失效的情况，分析其原理并记录解决方案

## 场景还原

#### 修复前的代码：

```java
if (FI_FIRST_STATUS_FOLLOW_UP.getCode().equals(current)) {
	this.explanation(orderId);
}
```

```java
@CustomerOrderNotify(tag = "ASSIST_PURCHASE", id = ORDER_ID)
@OrderEventStatus(methodName = "explanation", status = "wbStatus", orderTag = OrderTag.orderId)
@OrderEvent(tags = {ORDER, USER})
@Override
public void explanation(Integer orderId) {
	status(orderId, OrderStatus.ASSIST_PURCHASE, WorkbenchStatus.FOLLOW_UP);
}
```

> 这里explanation方法的所有切面都失效了

## 原理分析

### Spring AOP的实现原理

Spring AOP是基于动态代理机制实现的，通过动态代理（默认是jdk动态代理）生成对象的代理对象，当外部调用目标对象的相关方法时，Spring注入的其实是代理对象的Proxy，通过调用代理对象的方法执行AOP增强处理，然后回调目标对象的方法。

在一个对象的内部调用其他方法时，并没有产生代理对象，而是当前对象自己在执行，没有代理对象，自然也就没有办法执行增强的逻辑。

可以理解为，代理对象就像是一个包装公司，通过创建代理对象执行逻辑就相当于包装公司帮我们执行，并且进行一系列的包装，而内部调用则是我们亲历亲为，不借助包装公司，自然也就没有美化的功能。

## 解决方案

明白了上面的原理，自然也就容易想到解决方法，我们只需要在内部调用的时候，不使用当前对象（this），而是获取到它的代理对象即可正常的触发AOP的相关功能。

1. 在SpringBoot的启动类上开启暴露代理对象
2. 在具体代码中获取代理对象执行逻辑

具体代码如下：

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```

```java
// SpringAOP内部调用时会失效，所以获取该对象的代理，来执行方法
IOrderEventService currentProxy = (IOrderEventService) AopContext.currentProxy();
if (FI_FIRST_STATUS_FOLLOW_UP.getCode().equals(current)) {
	currentProxy.explanation(orderId);
}
```
如果是配置文件的形式，可以在spring-context.xml中配置暴露代理对象
```xml
<aop:aspectj-autoproxy proxy-target-class="true" expose-proxy="true"/>
```

---

#### 参考文章：

https://www.jianshu.com/p/f6b539a36b93