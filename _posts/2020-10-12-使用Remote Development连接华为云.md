
## 使用vscode 的remote 远程连接华为云

### 前言
博主前阵子，买了个`华为云的HECS(云耀云服务器)`,闲置严重，决定当作宿主机学习一下Linux编程的相应知识。ps(华为云也有自己的在线ide，不过还是自己的香....)

开发环境
> 本地win10+vscode，远程Linux

### remote development

在vscode的插件仓库搜索`remote development`，安装后，`ctrl + shfit +p`，输入**remote-ssh:add-new ssh hosts**

之后输入ssh 远程连接的命令,
>ssh 用户名@ip地址

之后连接即可，其中会让人输入远程登录的密码。

具体可以参考
[vscode官方文档](https://code.visualstudio.com/docs/remote/ssh)
[VScode远程调试remote development](https://blog.csdn.net/lkangkang/article/details/89892234)

### 免密码登录

相当于将本机(win10)的`id_rsa.pub (公钥)`内容加载到宿主机(linux) 的`authorized_keys`

两台Linux主机直接可以直接用
>ssh-copy-id -i ~/.ssh/id_rsa.pub 用户名@ip地址

但是win10下，行不通。简单点win10直接记事本打开`id_rsa.pub`(在c盘的用户目录下有个隐藏ssh文件，如果没有可以`ssh-keygen`生成，最好有用`xshell`连接的经验)，将里面的内容复制到
`/root/.ssh/authorized_keys` 既可以完成。

可以参考[SSH的免密登录详细步骤（注释+命令+图)](https://blog.csdn.net/SXY16044314/article/details/90605069),看看

### 使用vscode 远程编写一个c函数

`ctrl + shfit +p` 输入**remote-ssh:connect to hosts**,当然也可以直接单击vscode左下方的图标，选择Linux界面。

之后就会弹出一个新的vscode 界面，和本地正常的操作几乎一样！

- 新建hello.c文件
- 输入代码
```
#include <stdio.h>
main() {
  printf("hello world\n");
}
```
- `ctrl+s`保存，`` ctrl +shfit+` ``新建终端

- cd xxx 进入代码的所在目录
- gcc hello.c -o hello &&./hello

**hello world**
