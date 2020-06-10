# Java反射常用API汇总

## **一、类对象的获取**

1.通过对象获取

```java
Object obj = new Object();
obj.getClass();
```

2.通过类名获取

```java
Object.class;
```

3.通过类的路径名获取

```java
Class.forName("com.metadata.Student");
```

 

## **二、类的实例化和构造函数**

获取到的class对象可以直接通过clazz.newInstance()方法实例化，但是需要目标类有默认无参构造函数，不然会抛出异常。

在类没有默认无参构造函数，或者需要某个具体的构造函数来实例化的情况，需要通过Constructor类的newInstance()来完成。

 

1.获取公有构造函数，不包括父类

```java
//Classpublic Constructor<?>[] getConstructors() 
public Constructor<T> getConstructor(Class<?>... parameterTypes)
```

2.获取**当前类**构造函数，忽略修饰符

```java
//Class
public Constructor<?>[] getDeclaredConstructors()
public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)
```

构造函数调用

```java
//Constructor
public T newInstance(Object... initargs)
//忽略修饰符，强制调用
public void setAccessible(boolean flag)
```

 

## **三、类成员变量的获取**

1.获取公有变量，包括父类

```java
//Class
public Field[] getFields()
public Field getField(String name)
```

2.获取**当前类**成员变量，忽略修饰符

```java
//Class
public Field[] getDeclaredFields()
public Field getDeclaredField(String name)
```

成员变量赋值

```java
//Field
//obj为实例对象
public void set(Object obj,Object value)
//忽略修饰符，强制调用
public void setAccessible(boolean flag)
```

 

## **四、类方法的获取**

1.获取公有方法，包括父类

```java
//Class
public Method[] getMethods()
public Method getMethod(String name,
                        Class<?>... parameterTypes)
```

2.获取**当前类**方法，忽略修饰符

```java
//Class
public Method[] getDeclaredMethods()
public Method getDeclaredMethod(String name,
                                Class<?>... parameterTypes)
```

方法调用

```java
//Method
//obj为类实例化对象，如果为静态方法obj为Null
invoke(Object obj, Object... args)
//忽略修饰符，强制调用
public void setAccessible(boolean flag)
```