# Typora+PicGo+腾讯云COS实现图片上传[转载]


[TOC]

## 一、前言

众所周知，Typora是一款极佳的md文档编写工具，平时我们写md文件或多或少都会插入一些图片，但是由于markdown文件只是纯文本文件，因此当您嵌入图像时，md文件不会“拥有”这些图像，而只是对使用的外部图像文件保持弱引用。当您移动或共享这些文件时，这些图像也应被移动或共享，这带来了维护成本。但是，如果这些图像在线托管，则可以自由移动或共享文件，而无需在纯文本和所用图像之间保持引用。

Typora现在支持iPic，uPic，PicGo等应用程序，这类应用程序可以将图像上传到Imgur，七牛云，腾讯云COS，Github或其他图像托管服务。所以今天我们就来讲述一下利用腾讯云COS来实现Typora在线图床。至于其他的可以类比！

## 二、安装Typora和PicGo

### 1、Typora安装：

因为在较新版本的Typora中（在MacOS上为0.9.9.32或在Windows / Linux上为0.9.84），才有“上传图像”功能，这样才可以通过第三方应用程序或脚本将图像上传到云图像存储，所以Typora必须升级到比较新的版本（我这里是0.9.86(beta)）。安装过程较为简单，此处省略。

### 2、PicGo安装：

Picgo是一款图床工具，就是自动把本地图片转换成链接的一款工具，网络上有很多图床工具，就目前使用种类而言，PicGo算得上一款比较优秀的图床工具。
而且我们可以看到这里Typora是支持PicGo的。

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132525.png)

下载地址：https://github.com/Molunerfinn/PicGo/releases
考虑到通过这个github的网址下载可能会非常慢，我这里提供网盘链接：https://pan.baidu.com/s/1mHG1KXZtq0kzOqtEPFcl1Q
提取码：b4kv

安装成功后，整个要使用的工具都已经弄好，准备下一步相关配置！

### 三、腾讯云COS创建对象存储

进入腾讯云对象存储，在存储桶列表中点击创建存储桶

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132550.png)

设置一些存储桶的配置信息

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132558.png)

最后还有一个点，密匙管理

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132610.png)

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132618.png)

没有的话点击新建密钥，有的话就可以跳过这步，出现下面则表明腾讯云COS相关工作完成。（友情提示，这个时候不要关闭腾讯云COS，因为等下还要用到刚刚我们配置的这些信息）

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132627.png)

### 四、配置Typora和PicGo

1、打开PicGo，图片上传选择腾讯云COS

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132635.png)

2、然后就要配置PicGo上传服务

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132642.png)

如果不知道的话，也可以点进去看出存储桶基本配置以及点进访问密匙查看APPID，密匙等等。

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132649.png)

![在这里插入图片描述](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200520132657.png)这里我们PicGo跟腾讯云COS之间的上传服务就设置好了，你可以测试一下，上传一张图片就可以看到腾讯云COS上就有了，速度很快。（如果失败的话，你重新检查上面这几步，按照教程来做是绝对没有问题的）

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132726.png)

3、关联Typora与PicGo，打开Typora 文件->偏好设置

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132740.png)

找到图像上传设置，配置如下：

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132747.png)

最后设置完成我们验证一下

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132753.png)

去到腾讯云COS也可以看到我们刚刚测试上传的图片

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132800.png)

### 五、Typora图片上传测试

不管是剪切板直接粘贴图片到文本还是正常的图片插入，都可以

p.s忘了一点，之前PicGo这里一定要是md格式的：

![在这里插入图片描述](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200518153611957.png)

### 六、可能遇到的问题

1、上传图片失败。错误信息：{“success”：false}
出现这个问题的可能，就是有时候文件名冲突了，比如你上传过一张img.jpg的图片，再上传名称一样的图片就会失败，解决办法也很简单，打开picgo设置，将时间戳重命名打开 。不过我一般没怎么打开，我比较不太喜欢用这种时间命名图片。当然萝卜青菜各有所爱，这种出现这个问题的话，办法只能是这样。（不打开的话，注意不要重复上传即可）

![在这里插入图片描述](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520132824.png)

2、其他问题暂时还没有遇到，如果你有遇到的话可以提出来

### 七、总结

现在Typora做的越来越好了，图片上传功能也尽量实现简单化了，这篇文章就这讲述了Typora配合在线图床写md文档的方法，至于其他的图床，其实和这差不多。如果大家还想了解更多或者想要改进的一些功能的话也可以去Typora在Github上的issue仓库 点这

（插一句，Github和Gitee搭建图床不要钱，但是选我腾讯云COS的原因就是因为它超级快，而且腾讯云COS经常有优惠很便宜）
————————————————
版权声明：本文为CSDN博主「Aledsan」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_43465312/article/details/106191126

