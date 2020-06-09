# MyBatis的xml文件中resultType如何引用一个内部类

场景，将一个sql写到xml文件中时，resultType直接复制了内部类的全限定类名，结果报错了

![image-20200605145218179](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200605145226.png)

解决方法：

将点换成美元符号即可，表示内外关系

另外，使用内部类作为返回值必须具备以下条件

1. 内部类必须有无参构造函数
2. 内部类必须为静态类