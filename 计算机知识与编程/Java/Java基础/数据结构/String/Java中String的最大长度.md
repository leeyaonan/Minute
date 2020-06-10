# Java中String的最大长度

>https://mp.weixin.qq.com/s/AOqHr_kRe2eaRZivmnJ2zQ

String是Java中很重要的一个数据类型，除了基本数据类型以外，String是被使用的最广泛的了，但是，关于String，其实还是有很多东西容易被忽略的。

就如本文我们要讨论的问题：Java中的String有没有长度限制？

这个问题要分两个阶段看，分别是编译期和运行期。不同的时期限制不一样。

## 编译期

首先，我们先来合理的推断一下，当我们在代码中使用`String s = "";`的形式来定义String对象的时候，""中字符的个数有没有限制呢？

既然是合理的推断，那就要要足够的依据，所以我们可以从String的源码入手，根据public `String(char value[], int offset, int count)`的定义，count是int类型的，所以，char value[]中最多可以保存`Integer.MAX_VALUE`个,即`2147483647`字符。(jdk1.8.0_73)

但是，实验证明，String s = "";中，最多可以有65534个字符。如果超过这个个数。就会在`编译期`报错。

```java
public static void main(String[] args) {

    String s = "a...a";// 共65534个a
    System.out.println(s.length());

    String s1 = "a...a";// 共65535个a
    System.out.println(s1.length());
}
```

以上代码，会在String s1 = "a…a";// 共65535个a处编译失败：

```
✗ javac StringLenghDemo.java
StringLenghDemo.java:11: 错误: 常量字符串过长
```

**明明说好的长度限制是2147483647，为什么65535个字符就无法编译了呢？**

当我们使用字符串字面量直接定义String的时候，是会把字符串在`常量池`中存储一份的。那么上面提到的65534其实是常量池的限制。

常量池中的每一种数据项也有自己的类型。Java中的UTF-8编码的Unicode字符串在常量池中以`CONSTANT_Utf8`类型表示。

`CONSTANTUtf8info`是一个CONSTANTUtf8类型的常量池数据项，它存储的是一个常量字符串。常量池中的所有字面量几乎都是通过CONSTANTUtf8info描述的。CONSTANTUtf8_info的定义如下：

```
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```

由于本文的重点并不是CONSTANTUtf8info的介绍，这里就不详细展开了，我们只需要我们使用字面量定义的字符串在class文件中，是使用CONSTANTUtf8info存储的，而CONSTANTUtf8info中有`u2 length`;表明了该类型存储数据的长度。

u2是`无符号的16位整数`，因此理论上允许的的最大长度是2^16=65536。而 java class 文件是使用一种变体UTF-8格式来存放字符的，`null 值`使用两个 字节来表示，因此只剩下 65536－ 2 ＝ 65534个字节。

关于这一点，在the class file format spec中也有明确说明：

```
The length of field and method names, field and method descriptors, and other constant string values is limited to 65535 characters by the 16-bit unsigned length item of the CONSTANTUtf8info structure (§4.4.7). Note that the limit is on the number of bytes in the encoding and not on the number of encoded characters. UTF-8 encodes some characters using two or three bytes. Thus, strings incorporating multibyte characters are further constrained.
```

也就是说，在Java中，**所有需要保存在常量池中的数据，长度最大不能超过65535**，这当然也包括字符串的定义咯。

## 运行期

上面提到的这种String长度的限制是编译期的限制，也就是使用String s= "";这种字面值方式定义的时候才会有的限制。

那么。String在运行期有没有限制呢，答案是有的，就是我们前文提到的那个`Integer.MAX_VALUE `，这个值约等于4G，在运行期，如果String的长度超过这个范围，就可能会抛出异常。(在jdk 1.9之前）

int 是一个 32 位变量类型，取正数部分来算的话，他们最长可以有

```shell
2^31-1 =2147483647 个 16-bit Unicodecharacter

2147483647 * 16 = 34359738352 位
34359738352 / 8 = 4294967294 (Byte)
4294967294 / 1024 = 4194303.998046875 (KB)
4194303.998046875 / 1024 = 4095.9999980926513671875 (MB)
4095.9999980926513671875 / 1024 = 3.99999999813735485076904296875 (GB)
```

有近 4G 的容量。