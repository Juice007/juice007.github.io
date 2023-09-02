---
layout: post
title:  "git中使用ssh认证拉取代码"
date:   2023-08-24 11:05:45 +0800
categories: git
---

git中拉取代码有两种方式，一种是https的方式，一种是ssh的方式。两种方式都是为了认证鉴权，主要区别还是在于加密与传输的方式上有所不同。孰优孰劣，我个人认为并无定论。

总体来讲，https的方式更简便。笔者在尝试了两种方式后，最终还是决定推荐使用https的方式拉取代码。

## 一、https方式

https方式主要通过用户名和密码来进行认证，拉取代码时只需要输入代码托管平台的username和password即可。如果是Mac电脑，一般会有保存到 Keychain 的选项，以后可以不用重复输入。
操作简单，这里不做过多介绍。

## 二、ssh方式

ssh协议使用上比起https复杂一些，这里重点介绍下操作流程：

1、生成本机公钥（以下两种算法任选其一）

``` shell
# 1、使用rsa算法生成公钥
ssh-keygen -t rsa -C "your_email@example.com"
# 2、使用ed25519算法生成公钥
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2、查看本机公钥

``` shell
# 查看所有密钥
ls -al ~/.ssh
# 打印本机公钥
cat ~/.ssh/id_rsa.pub
```

3、到对应的代码托管网站添加公钥
常用的代码托管网站，如：github、gitlab等添加公钥的位置一般都在个人-设置里，以github为例

4、验证

``` shell
ssh -T git@github.com
```

## 三、使用ssh可能遇到的问题

1、验证时报错：  
REMOTE HOST IDENTIFICATION HAS CHANGED!  
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!  
Someone could be eavesdropping on you right now (man-in-the-middle attack)!

![image](/assets/img/2023/image.png)

这种情况，需要先查看下known_hosts文件

``` shell
code ~/.ssh/known_hosts
```

使用VSCode打开known_hosts文件，如果你没有安装code命令，可以从Finder进入目录后打开检查github的私钥有没有配置进去或者有没有配置对
![image](/assets/img/2023/image%202.png)
如果不对，你可以删除掉github.com开头的代码行，然后重新执行

``` shell
ssh -T git@github.com
```

会自动重新生成好对应的代码

![Alt text](/assets/img/2023/image%203.png)

如图中，输入yes后就会自动重新生成

## 四、一些经验&感悟

1、如果你在尝试的过程中，还遇到了其他报错，建议删除掉`.ssh`文件下的所有文件，重新生成一下。

2、即使主工程使用了SSH的方式认证，在执行`pod install`的时候一些依赖项可能也还是需要输入账户名和密码，所以还不如一开始就使用HTTPS的方式，然后把账户名、密码存到keychain里，以后也不用再输入。

3、如果你电脑上有多个平台，建议使用不同的算法生成公钥
![Alt text](/assets/img/2023/image%204.png)

附：参考资料

<https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent>
