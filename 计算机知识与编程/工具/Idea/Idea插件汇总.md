# IDEA插件汇总（转载+整合）

[TOC]



当前使用的IDEA版本是2020.1。随着IDEA版本的升级，有些插件不再支持，而有些插件变成了收费插件，这些插件将不再推荐。以下列举的，都是亲测可以在2020.1版本的IDEA中使用的插件。

## google-java-format 代码自动格式化



简介：google-java-format插件可以帮助我们不通过对应的快捷键就可以实现特定方式下自动格式化代码。

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520143941.png)



## Translation



简介：翻译插件，支持google翻译、百度翻译、有道翻译。

使用：快捷键Ctrl + Shift + O



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520143944.png)





## Key promoter X



简介：Key Promoter X 是一个提示插件。在IDEA里使用鼠标操作时，会有这个操作的快捷键在界面的右下角进行告知。有个小缺点是有些没有快捷键的操作，会直接把操作的名字提示出来，实际上那样的提示是没有作用的，可以点击Don't show again来忽略。

对了，我把公众号 Java后端 发布过的 IDEA 相关文章整理成了 PDF ,需要的关注 Java后端，然后回复 666 下载。



![img](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhwEf5tY1GPtxwHibh6zcUKPLdJbybglibt5ibUgdwdyHH4aEgTNhuIwCaLPJnZKjg3ib6ibn852LFExNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## Alibaba Java Coding Guidelines



简介：阿里巴巴代码规范检测。不符合代码规范的地方会有波浪线，鼠标移上去就会有相应的提示，有些问题甚至可以快速修复。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520143951.png)



## Leetcode Editor



简介：LeetCode插件，可以在IDEA中在线刷题。上班摸鱼属实方便，表面上我在干活，实际上我在刷算法题。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520143952.png)



## RestfulToolkit



简介：搜索URL，准确的说是搜索SpringMVC项目里，Controller层的@RequestMapping里的URL，通过URL匹配到相应的Controller层方法。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520143955.png)



使用：快捷键：Ctrl + \ 或Ctrl + Alt + N



## Jclasslib Bytecode Viewer



简介：看类的字节码文件。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520143958.png)



## CamelCase



简介：在几种字符串格式之间来回切换。有一下几种格式：



- 驼峰，第一个单词首字母小写，其他单词首字母大写
- 所有字母小写，单词间下划线分隔
- 所有字母小写，单词间空格分隔
- 所有字母小写，单词间短横线分隔
- 每个单词首字母全部大写
- 所有字母大写，单词间下划线分隔



使用：按住Shift + Alt再不停的按U，会把选中内容的单词的下划线转驼峰转大写等，不停的转换，直到你想要的。

## Jrebel for Intellij



简介：JRebel是一款JVM插件，它使得Java代码修改后不用重启系统，立即生效。当然还是需要按一下快捷键才能生效的。

可以自己写代码，或者找一个在线网站生成一个guid。然后打开插件激活界面，输入Team URL https://jrebel.qekang.com/网上随便生成的一个guid（网上有一些说用http://127.0.0.1:8888，我试了一下发现不行），然后输入自己的邮箱。点击下方的Change license按钮，激活成功。



## String Manipulation



功能：变量名使用驼峰形式、常量需要全部大写等等，编码解码等等。总的来说就是对字符串的处理。



使用：选中需要处理的内容后，按快捷键Alt + M，即可弹出工具功能列表。后面的具体功能也可以使用相应的数字或字母，而不需要鼠标点击。



![img](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhwEf5tY1GPtxwHibh6zcUKPALJvYcSkKYho5SoDAdYkXr3RZT72qLCMRt5iaKNlru42DeQ7KGvZwBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## Free Mybatis Plugin



简介：可以通过mapper接口里的方法跳转到mapper.xml里。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520144003.png)



## SequenceDiagram



简介：



- 生成简单序列图。
- 单击图形形状来导航代码。
- 从图中删除类。
- 将图表导出为图像。
- 通过“设置”>“其他设置”>“序列”从图表中排除类





使用：光标定位在方法名或者方法体内，在右键菜单里选择Sequence Diagram。然后可以填方法的调用深度，默认是5.

![img](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhwEf5tY1GPtxwHibh6zcUKPSFwfmwU7aYeZHeibyFVzDN8YqQhd6gdqhDx2CfjyCwkoBlNBwo3nYsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图里面不仅有自己写的方法，还有调用的其他第三方库的方法，看着比较杂乱。



## GenerateAllSetter



简介：可以直接生成这个对象的所有set方法，非常方便。



使用：将光标放在变量声明的那一行，注意不能是分号后面。然后按快捷键Alt + Enter，就会弹出菜单供你选择。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520144010.png)

## Chinese(Simplified) Language Pack EAP



简介：2020.1版本开始支持的官方汉化插件。2020-04-10的使用情况来看，插件还存在缺陷，有些地方显示的不是汉化而是一段代码。可以等这个插件再完善一些再使用。



## Rainbow Brackets



简介：彩虹括号。括号嵌套时，会用不同的颜色将括号标出。光标移到一个括号上，配对的括号也会高亮显示。

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520144012.png)



## IDEA QAPlug



简介：帮助我们提前找到潜在的问题bug



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520144015.png)



静静等待一会,在下方生成分析结果。



![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520144018.png)