##IntelliJ IDEA github使用



1. <a href="#git介绍与安装">git介绍与安装</a>
2. <a href="#Intellij使用git">Intellij使用git</a>



#####【<a name="git介绍与安装" id="git介绍与安装" ><font color=black>git介绍与安装</font></a>】

官网:https://github.com/    
总结的很好的git资料：http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000   
在Linux上安装Git:
```shell
$ git  #提示git没有安装，还会告诉你如何安装git
$ sudo apt-get install git #安装
$ git version #查看版本
```

#####【<a name="Intellij使用git" id="Intellij使用git" ><font color=black>Intellij使用git</font></a>】

打开Intellij软件，在菜单栏 VCS-->Checkout from Version Control-->GitHub   
报错时，可能是没安装git，或者是没配置git路径。在菜单栏 File-->Settings...-->Version Control-->Git   
Path to Git executable:/usr/bin/git(填写git安装软件路径,使用命令：whereis git可以看到安装的路径)   
-->GitHub中:Host:github.com输入github用户名和密码，这样下载项目时，就不用每次提示要输入密码了。
