# Window下git生成SSH Key

2018.01.16 15:46:15字数 407阅读 4509
https://www.jianshu.com/p/95262f5eba7a
原文地址：http://blog.csdn.net/lsyz0021/article/details/52064829
当我们使用github或者bitbucket等仓库时我们有可能需要ssh认证，所以需要生成他的ssh key。
1、首先你要安装git工具
下载地址：https://git-scm.com/downloads
2、右键鼠标，选中 “Git Bash here”，当然你也可以在windows的 “开始”--->“所以程序”，或者安装目录打开它

3、输入指令，进入.ssh文件夹
cd ~/.ssh/

image
如果提示 “ No such file or directory”，你可以手动的创建一个 .ssh文件夹即可
命令为：
mkdir ~/.ssh
4、配置全局的name和email，这里是的你github或者bitbucket的name和email
git config --global user.name "你的用户名"
git config --global user.email "你的公司或个人邮箱"
5、生成key
ssh-keygen -t rsa -C "你的公司或个人邮箱"
连续按三次回车，这里设置的密码就为空了，并且创建了key。

Your identification has been saved in /User/Admin/.ssh/id_rsa.
Your public key has been saved in /User/Admin/.ssh/id_rsa.pub.
The key fingerprint is:
………………
最后得到了两个文件：id_rsa和id_rsa.pub
6、打开Admin目录进入.ssh文件夹，用记事本打开id_rsa.pub，复制里面的内容添加到你github或者bitbucket ssh设置里即可

image

image
这是bitbucket的添加key，点击右上方的头像，选择设置，然后

image
这是github添加key

image
7、测试是否添加成功 ----------可以忽略
bitbucket输入命令：
ssh -T git@bitbucket.org
提示：“You can use git or hg to connect to Bitbucket. Shell access is disabled.” 说明添加成功了
github输入命令：
ssh git@github.com
提示：“Hi lsyz0021! You've successfully authenticated, but GitHub does not provide shel l access.”说明添加成功。