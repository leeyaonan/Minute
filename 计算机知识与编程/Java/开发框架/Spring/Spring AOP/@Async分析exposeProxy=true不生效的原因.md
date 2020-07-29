# @Async分析exposeProxy=true不生效原因

> https://blog.csdn.net/Sophisticated_/article/details/102793703

#### 前言

本文标题包含有`'靓丽'`的字眼：`Spring框架bug`。相信有的小伙伴心里小九九就会说了：又是一篇标题党文章。
鉴于此，此处可以很负责任的对大伙说：本人**所有文章**绝不`哗众取宠`，除了干货只剩干货。

> 相信关注过我的小伙伴都是知道的，我只递送干货，绝不标题党来浪费大家的时间和精力~那无异于`谋财害命`(说得严重了，不喜勿喷)
> 关于标题党的好与坏、优与劣，**此处我不置可否**

本篇文章能让你知道`exposeProxy=true`真实作用和实际作用范围，从而能够在开发中更精准的使用到它。

#### 背景

本来一切本都那么平静，直到我用了`@Async`注解，好多问题都接踵而至（上篇文章已经解决大部分了）。在上篇文章中，为了解决`@Async`同类方法调用问题我提出了两个方向的解决方案：

1. 自己注入自己，然后再调用接口方法（当然此处的一个变种是使用编程方式形如：

   `AInterface a = applicationContext.getBean(AInterface.class);`

   这样子手动获取也是可行的~~~本文不讨论这种比较直接简单的方式）

2. 使用`AopContext.currentProxy();`方式

方案一上篇文章已花笔墨重点分析，毕竟方案一我认为更为重要些。本文分析使用方案二的方式，它涉及到AOP、代理对象的暴露，因此我认为本文的内容对你平时开发的影响是不容小觑，可以重点浏览咯~

**我相信绝大多数小伙伴都遇到过这个异常：**

```java
 java.lang.IllegalStateException: Cannot find current proxy: Set 'exposeProxy' property on Advised to 'true' to make it available.
	at org.springframework.aop.framework.AopContext.currentProxy(AopContext.java:69)
	at com.fsx.dependency.B.funTemp(B.java:14)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:343)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:206)
	at com.sun.proxy.$Proxy44.funTemp(Unknown Source)
	...
```

然后当你去靠**度娘**搜索解决方案时，发现`无一例外`都教你只需要这么做就成：

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```

本文我想说的可能又是一个`技术敏感性`问题，其实绝大多数情况下你按照这么做是可行的，直到你遇到了`@Async`也需要调用本类方法的时候，你就有点绝望了，然后本文或许会成为了你的**救星~**

本以为加了`exposeProxy = true`就能顺风顺水了，但它却出问题了：依旧报如上的异常信息。如果你看到这里也觉得`不可思议`，那么本文就更能体现它的价值所在~

> 此问题我个人把它归类为Spring的bug我觉得是无可厚非的，因为它的语义与实际表现出来的结果想悖了，so我把定义为**Spring框架的bug**。
> 对使用者来说，标注了`exposeProxy = true`，理论上就应该能够通过`AopContext.currentProxy()`拿到代理对象，可惜Spring这里却掉链子了，有点名不副实之感~

## 示例

本文将以多个示例来模拟不同的使用case，首先从直观的结果上先了解`@EnableAspectJAutoProxy(exposeProxy = true)`的作用以及它存在的问题。

备注：下面所有示例都建立在`@EnableAspectJAutoProxy(exposeProxy = true)`已经开启的前提下，形如：

```java
@Configuration
@EnableAspectJAutoProxy(exposeProxy = true) // 暴露当前代理对象到当前线程绑定
public class RootConfig {
}
```

#### 示例一

```java
@Service
public class B implements BInterface {

    @Transactional
    @Override
    public void funTemp() {
        ...

        // 希望调用本类方法  但是它抛出异常，希望也能够回滚事务
        BInterface b = BInterface.class.cast(AopContext.currentProxy());
        System.out.println(b);
        b.funB();
    }

    @Override
    public void funB() {
        // ... 处理业务属于  
        System.out.println(1 / 0);
    }
}
```

结论：能正常work，事务也会生效~

#### 示例二

同类内方法调用，希望异步执行被调用的方法（希望`@Async`生效）

```java
@Service
public class B implements BInterface {
    @Override
    public void funTemp() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
        // 希望调用本类方法  但是希望它去异步执行~
        BInterface b = BInterface.class.cast(AopContext.currentProxy());
        System.out.println(b);
        b.funB();
    }
    @Async
    @Override
    public void funB() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
    }
}
```

结论：执行即报错

```java
java.lang.IllegalStateException: Cannot find current proxy: Set 'exposeProxy' property on Advised to 'true' to make it available.
```

#### 示例三

同类内方法调用，希望异步执行被调用的方法，**并且**在入口方法处使用事务

```java
@Service
public class B implements BInterface {
    @Transactional
    @Override
    public void funTemp() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
        // 希望调用本类方法  但是希望它去异步执行~
        BInterface b = BInterface.class.cast(AopContext.currentProxy());
        System.out.println(b);
        b.funB();
    }
    @Async
    @Override
    public void funB() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
    }
}
```

结论：正常work没有报错，`@Async异步生效、事务也生效`

#### 示例四

和`示例三`的唯一区别是把事务注解`@Transactional`标注在被调用的方法处（和`@Async`同方法）：

```java
@Service
public class B implements BInterface {
    @Override
    public void funTemp() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
        // 希望调用本类方法  但是希望它去异步执行~
        BInterface b = BInterface.class.cast(AopContext.currentProxy());
        System.out.println(b);
        b.funB();
    }
    @Transactional
    @Async
    @Override
    public void funB() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
    }
}
```

结论：同示例三

#### 示例五

把`@Async`标注在入口方法上：

```java
@Service
public class B implements BInterface {
    @Transactional
    @Async
    @Override
    public void funTemp() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
        BInterface b = BInterface.class.cast(AopContext.currentProxy());
        System.out.println(b);
        b.funB();
    }
    @Override
    public void funB() {
        System.out.println("线程名称：" + Thread.currentThread().getName());
    }
}
```

结论：请求即报错

```java
java.lang.IllegalStateException: Cannot find current proxy: Set 'exposeProxy' property on Advised to 'true' to make it available.
	at org.springframework.aop.framework.AopContext.currentProxy(AopContext.java:69)
12
```

#### 示例六

偷懒做法：直接在实现类里写个方法（public/private）然后注解上`@Async`

> 我发现我司同事有大量这样的写法，所以专门拿出作为示例，以儆效尤~

```java
@Service
public class B implements BInterface {
	...
    @Async
    public void fun2(){
        System.out.println("线程名称：" + Thread.currentThread().getName());
    }
}
```

结论：因为方法不在接口上，因此**肯定无法通过获取代理对象调用它**。

> 需要注意的是：即使该方法不属于接口方法，但是标注了`@Async`所以最终生成的还是B的代理对象~(哪怕是private访问权限也是代理对象)

可能有的小伙伴会想通过`context.getBean()`获取到具体实现类再调用方法行不行。咋一想可行，实际则不是不行的。
这里**再次强调一次**，若你是AOP是JDK的动态代理的实现，这样100%报错的：

```java
BInterface bInterface = applicationContext.getBean(BInterface.class); // 正常获取到容器里的代理对象
applicationContext.getBean(B.class); //报错  NoSuchBeanDefinitionException
// 原因此处不再解释了，若是CGLIB代理，两种获取方式均可~
```

> 备注：虽说`CGLIB`代理方式用实现类方式可以获取到代理的Bean，但是**强烈不建议**依赖于代理的具体实现而书写代码，这样移植性会非常差的，而且接手的人肯定也会一脸懵逼、二脸懵逼…

因此当你看到你同事就在本类写个方法标注上`@Async`然后调用，请制止他吧，做的无用功~~~（**关键自己还以为有用，这是最可怕的深坑~**）

## 原因大剖析

**找错的常用方法：逆推法。**
首先我们找到报错的最直接原因：`AopContext.currentProxy()`这句代码报错的，因此有必要看看`AopContext`这个**工具类**：

```java
// @since 13.03.2003
public final class AopContext {
	private static final ThreadLocal<Object> currentProxy = new NamedThreadLocal<>("Current AOP proxy");
	private AopContext() {
	}
	// 该方法是public static方法，说明可以被任意类进行调用
	public static Object currentProxy() throws IllegalStateException {
		Object proxy = currentProxy.get();
		// 它抛出异常的原因是当前线程并没有绑定对象
		// 而给线程版定对象的方法在下面：特别有意思的是它的访问权限是default级别，也就是说只能Spring内部去调用~
		if (proxy == null) {
			throw new IllegalStateException("Cannot find current proxy: Set 'exposeProxy' property on Advised to 'true' to make it available.");
		}
		return proxy;
	}
	// 它最有意思的地方是它的访问权限是default的，表示只能给Spring内部去调用~
	// 调用它的类有CglibAopProxy和JdkDynamicAopProxy
	@Nullable
	static Object setCurrentProxy(@Nullable Object proxy) {
		Object old = currentProxy.get();
		if (proxy != null) {
			currentProxy.set(proxy);
		} else {
			currentProxy.remove();
		}
		return old;
	}
}
```

从此工具源码可知，决定是否抛出所示异常的直接原因就是请求的时候`setCurrentProxy()`方法是否被调用过。通过寻找发现只有两个类会调用此方法，并且都是Spring内建的类且都是代理类的**处理类**：`CglibAopProxy`和`JdkDynamicAopProxy`

> 说明：本文所有示例，都基于接口的代理，所以此处只以`JdkDynamicAopProxy`作为代表进行说明即可

我们知道在执行代理对象的目标方法的时候，都会交给`InvocationHandler`处理，因此做事情的在`invoke()`方法里：

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
	...
	@Override
	@Nullable
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		...
			if (this.advised.exposeProxy) {
				// Make invocation available if necessary.
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}
		...
		finally {
			if (setProxyContext) {
				// Restore old proxy.
				AopContext.setCurrentProxy(oldProxy);
			}
		}
	}
}
```

so，最终决定是否会调用set方法是由`this.advised.exposeProxy`这个值决定的，因此下面我们只需要关心`ProxyConfig.exposeProxy`这个属性值什么时候被赋值为true的就可以了。

> `ProxyConfig.exposeProxy`这个属性的默认值是false。其实最终调用设置值的是同名方法`Advised.setExposeProxy()`方法，而且是通过反射调用的

#### `@EnableAspectJAutoProxy(exposeProxy = true)`的作用

此注解它导入了`AspectJAutoProxyRegistrar`，最终`设置`此注解的两个属性的方法为：

```java
public abstract class AopConfigUtils {
	public static void forceAutoProxyCreatorToUseClassProxying(BeanDefinitionRegistry registry) {
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			definition.getPropertyValues().add("proxyTargetClass", Boolean.TRUE);
		}
	}
	public static void forceAutoProxyCreatorToExposeProxy(BeanDefinitionRegistry registry) {
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			definition.getPropertyValues().add("exposeProxy", Boolean.TRUE);
		}
	}
}
```

看到此注解标注的属性值最终都被设置到了`internalAutoProxyCreator`身上，也就是进而重要的一道菜：自动代理创建器。

在此各位小伙伴需要先明晰的是：`@Async`的代理对象并不是由自动代理创建器来创建的，而是由`AsyncAnnotationBeanPostProcessor`一个单纯的`BeanPostProcessor`实现的。

## 示例结论分析

本章节在掌握了一定的理论的基础上，针对上面的各种示例进行**结论性分析**。

#### 示例一分析

本示例目的是事务，可以参考开启事务的注解`@EnableTransactionManagement`。该注解向容器注入的是自动代理创建器`InfrastructureAdvisorAutoProxyCreator`，所以`exposeProxy = true`对它的代理对象都是生效的，因此可以正常work~

> 备注：`@EnableCaching`注入的也是自动代理创建器~so `exposeProxy = true`对它也是有效的

#### 示例二分析

很显然本例是执行`AopContext.currentProxy()`这句代码的时候报错了。报错的原因相信我此处不说，小伙伴应该个大概了。

`@EnableAsync`给容器注入的是`AsyncAnnotationBeanPostProcessor`，它用于给`@Async`生成代理，但是它仅仅是个`BeanPostProcessor`并不属于自动代理创建器，因此`exposeProxy = true`对它无效。
所以`AopContext.setCurrentProxy(proxy);`这个set方法肯定就不会执行，so但凡只要业务方法中调用`AopContext.currentProxy()`方法就铁定抛异常~~

#### 示例三分析

这个示例的结论，**相信是很多小伙伴都没有想到的**。仅仅只是加入了事务，`@Asycn`竟然就能够完美的使用`AopContext.currentProxy()`获取当前代理对象了。

为了便于理解，我分步骤讲述如下，不出意外你肯定就懂了：

1. `AsyncAnnotationBeanPostProcessor`在创建代理时有这样一个逻辑：若已经是`Advised`对象了，那就只需要把`@Async`的增强器添加进去即可。若不是代理对象才会自己去创建

```java
public abstract class AbstractAdvisingBeanPostProcessor extends ProxyProcessorSupport implements BeanPostProcessor {
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) {
		if (bean instanceof Advised) {
			advised.addAdvisor(this.advisor);
			return bean;
		}
		// 上面没有return，这里会继续判断自己去创建代理~
	}
}
```

1. 自动代理创建器`AbstractAutoProxyCreator`它实际也是个`BeanPostProcessor`，所以它和上面处理器的执行顺序很重要~~~
2. 两者都继承自`ProxyProcessorSupport`所以都能创建代理，且实现了`Ordered`接口
   \1. `AsyncAnnotationBeanPostProcessor`默认的order值为`Ordered.LOWEST_PRECEDENCE`。但可以通过`@EnableAsync`指定order属性来改变此值。 执行代码语句：`bpp.setOrder(this.enableAsync.<Integer>getNumber("order"));`
   \2. `AbstractAutoProxyCreator`默认值也同上。但是在把自动代理创建器添加进容器的时候有这么一句代码：`beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);` 自动代理创建器这个处理器是`最高优先级`
3. 由上可知因为标注有`@Transactional`，所以自动代理会生效，因此它会先交给`AbstractAutoProxyCreator`把代理对象生成好了，再交给后面的处理器执行
4. 由于`AbstractAutoProxyCreator`先执行，所以`AsyncAnnotationBeanPostProcessor`执行的时候此时Bean已经是代理对象了，由步骤1可知，此时它会沿用这个代理，只需要把切面添加进去即可~

从上面步骤可知，加上了事务注解，最终代理对象是由自动代理创建器创建的，因此`exposeProxy = true`对它有效，这是解释它能正常work的最为根本的原因。

#### 示例四分析

同上。

> `@Transactional`只为了创建代理对象而已，所在放在哪儿对`@Async`的作用都不会有本质的区别

#### 示例五分析

**此示例非常非常有意思，因此我特意拿出来讲解一下。**

咋一看其实以为是没有问题的，毕竟正常我们会这么思考：执行`funTemp()`方法会启动异步线程执行，同时它会把Proxy绑定在当前线程中，所以即使是新起的异步线程也有能够使用`AopContext.currentProxy()`才对。

但有意思的地方就在此处：**它报错了**，正所谓你以为的不一定就是你以为的。
解释：根本原因就是关键节点的执行时机问题。在执行代理对象`funTemp`方法的时候，绑定动作`oldProxy = AopContext.setCurrentProxy(proxy);`在前，目标方法执行（包括增强器的执行）`invocation.proceed()`在后。**so其实在执行绑定的还是在主线程里而并非是新的异步线程**，所以在你在方法体内（已经属于异步线程了）执行`AopContext.currentProxy()`那可不就报错了嘛~

#### 示例六分析

略。（上已分析）

## 解决方案

对上面现象原因可以做一句话的总结：**`@Async`要想顺利使用`AopContext.currentProxy()`获取当前代理对象来调用本类方法，需要确保你本Bean已经被自动代理创建器`提前代理`。**

> 在实际业务开发中：只要的类标注有`@Transactional`或者`@Caching`等注解，就可以放心大胆的使用吧

知晓了原因，解决方案从来都是信手拈来的事。
不过如果按照如上所说需要`隐式依赖`这种方案我**非常的不看好**，总感觉不踏实，也总感觉报错迟早要来。（**比如某个同学该方法不要事务了/不要缓存了，把对应注解摘掉就瞬间报错了**，到时候你可能哭都没地哭诉去~）

> 备注：`墨菲定律`在开发过程中从来都没有不好使过~~~程序员兄弟姐妹们应该深有感触吧

下面根据我个人经验，介绍一种解决方案中的最佳实践：

> 遵循的最基本的原则是：显示的指定比隐式的依赖来得更加的靠谱、稳定

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition(TaskManagementConfigUtils.ASYNC_ANNOTATION_PROCESSOR_BEAN_NAME);
        beanDefinition.getPropertyValues().add("exposeProxy", true);
    }
}
```

**这样我们可以在`@Async`和`AopContext.currentProxy()`就自如使用了，不再对别的啥的有依赖性~**

> 其实我认为最佳的解决方案是如下两个（都需要Spring框架做出修改）：
> 1、`@Async`的代理也交给自动代理创建器来完成
> 2、`@EnableAsync`增加`exposeProxy`属性，默认值给false即可（**此种方案的原理同我示例的最佳实践~**）

#### 总结

通过6组不同的示例，演示了不同场景使用`@Async`，并且对结论进行解释，不出意外，小伙伴们读完之后都能够掌握它的来龙去脉了吧。

最后再总结两点，小伙伴们使用的时候稍微注意下就行：

1. 请不要在异步线程里使用`AopContext.currentProxy()`
2. `AopContext.currentProxy()`不能使用在非代理对象所在方法体内