**具体步骤**

idea版本是2019.3

**一、开启IDEA的自动编译（静态）**

具体步骤：打开顶部工具栏 File -> Settings -> Build -> Compiler 然后勾选 Build project automatically 。

![img](D:\PersonalFiles\学习\笔记\Minute\计算机知识与编程\工具\Idea\Idea如何热部署SpringBoot\2020040714574426.png)

**二、开启IDEA的自动编译（动态）**

具体步骤：同时按住 Ctrl + Shift + Alt + / 然后进入Registry ，勾选自动编译并调整延时参数。

> compiler.automake.allow.when.app.running -> 自动编译
> compile.document.save.trigger.delay -> 自动更新文件

PS：网上极少有人提到compile.document.save.trigger.delay 它主要是针对静态文件如JS CSS的更新，将延迟时间减少后，直接按F5刷新页面就能看到效果！

![img](D:\PersonalFiles\学习\笔记\Minute\计算机知识与编程\工具\Idea\Idea如何热部署SpringBoot\2020040714574527.png)

**三、开启IDEA的热部署策略（非常重要）**

具体步骤：顶部菜单- >Edit Configurations->SpringBoot插件->目标项目->勾选热更新。

![img](D:\PersonalFiles\学习\笔记\Minute\计算机知识与编程\工具\Idea\Idea如何热部署SpringBoot\2020040714574628.png)

**四、在项目添加热部署插件（可选）**

温馨提示：

如果因为旧项目十分臃肿，导致每次都自动热重启很慢而影响开发效率，笔者建议直接在POM移除spring-boot-devtools依赖，然后使用Control+Shift+F9进行手工免启动快速更新！！

具体步骤：在POM文件添加热部署插件

```
<``dependency``>``  ``<``groupId``>org.springframework.boot`groupId``>``  ``<``artifactId``>spring-boot-devtools`artifactId``>``  ``<``scope``>runtime`scope``>`` ```dependency``>
```

打开谷歌浏览器，打开F12的Network选项栏，然后勾选【✅】Disable cache 。

![img](D:\PersonalFiles\学习\笔记\Minute\计算机知识与编程\工具\Idea\Idea如何热部署SpringBoot\2020040714574629.png)