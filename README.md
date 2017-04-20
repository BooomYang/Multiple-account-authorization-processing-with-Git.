###### Ps:使用场景未必广泛,更多的是透过此方式了解整个校验体系的必经环节，加深理解.
     

### 简单来讲,Git权限校验方式有两种,以Github为例: 
    
#### `HTTPS URLs`<br>
实际就是校验Github输入的账户和密码.
    
* Mac系统中:keychain会存储用户第一次输入的账户密码,下次访问仓库时自动读取,不必重复输入.<br>
* 非Mac:Git提供credential helper机制,下次访问时自动读取.
```c
$ git config --global credential.helper cache (输入的账号密码会在内存中缓存一段时间,默认15分钟)
$ git config --global credential.helper store (输入的账号密码会存储在 ~/.git-credentials中)   
```
#### `SSH URLs`
实际就是校验本地存储的私钥和添加至Github账户的公钥.     

在使用SSH校验之前,需要现在本地计算机中生成`SSH Keypair`.

* 生成SSH key步骤: 
```c
  1).  $ cd ~/.ssh  
       $ ls
```
      查看是否有`id_rsa`、`id_rsa.pub`,如果有可跳过步骤2.
```c
  2). $ ssh-keygen -t rsa -C "your_email@example.com"
```
      生成秘钥对. 
```c      
      Generating public/private rsa key pair.
      Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter] 
```
      这是提示生成路径. 默认按回车.
```c      
      Enter passphrase (empty for no passphrase): 
      Enter same passphrase again:
```
      提示是否输入passphrase密码,如果输入则push时需要输入此密码. <br>
      
      秘钥对创建好后,秘钥存储在本地(`~/.ssh/id_rsa`),公钥(`~/.ssh/id_rsa.pub`)应该添加到Github的账户.<br>
```c      
  3). 在Github上添加ssh key,即`id_rsa.pub`的内容拷贝进去. 
```  
```c  
  4). $ ssh -T git@github.com
```  
      测试该ssh,检测本地计算机与Github的连接状态. 
      
```c
    Permission denied (publickey).
```
      若出现`denied`,是因尚未将私钥(`id_rsa`)添加到`ssh-agent`中.
```c
     $ ssh-add ~/.ssh/id_rsa
     Identity added: /Users/yangwang/.ssh/id_rsa (/Users/yangwang/.ssh/id_rsa)
```     
      添加完成后,就可查看到当前ssh-agent存储的秘钥
```c
     $ ssh-add -l
     2048 SHA256:lw9meAZwbyZYTjXGTpY7G88k81LTPccBMZ/I2LT1bSI /Users/yangwang/.ssh/id_rsa (RSA)
```  
      再次检测与Github的连接状态
```c
     $ ssh -T git@github.com
     Hi BooomYang! You've successfully authenticated, but GitHub does not provide shell access.
```
### `同时使用多个Github账号`  

要想在一台机器上同时使用多个Github账号访问不同的仓库,该怎么做呢 ？ <br>
为方便演示,此时有两个不同的Github账号:
* `BooomYang` 
* `AllYang`  
<br>对应两个仓库: 
* `BooomYang/BoomRepo`
* `AllYang/AllYangRepo`


#### `SSH方案`
###### 前提 
两个账号均已经创建`SSH`秘钥对,且`SSH-key`已加入本地`ssh-agent`.
```c
   $ ssh-add -l
   2048 SHA256:lw9meAZwbyZYTjXGTpY7G88k81LTPccBMZ/I2LT1bSI /Users/yangwang/.ssh/id_rsa (RSA)
   2048 SHA256:WzSl1dqrCDe4WR2vq4KQi9EXcUHl76k00vhhwe5CgVk allYang_id_rsa (RSA)
```
在执行`pull`或`push`操作时，本地计算机`ssh-agent`与Github连接,进行权限校验.当本地计算机只有一个Github账号时,系统会采用这个唯一的账号去操作.当本地计算机有多个账号时,此时系统是无法区分判断的，只会有一个默认的账号,并使用这一个账号去操作所有的仓库,当不匹配时，则发生错误.    
###### 解决
关键就是手动配置文件`~/.ssh/config`.打开`~/.ssh/config`文件: 
```c
#BoomYang
Host BooomYang
     HostName github.com
     User git
     IdentityFile ~/.ssh/id_rsa

#AllYang
Host AllYang
     HostName github.com
     User git
     IdentityFile ~/.ssh/allYang_id_rsa

```
编辑为上图所示,对比一下两个项目的SSH URL: 
```c
git@github.com:BooomYang/BoomRepo.git
git@github.com:AllYang/AllYangRepo.git
```

分析一下:<br>
* `git`是本地`ssh-agent`与Github服务器建立`SSH`连接使用的用户名(`User`)
* github.com是Github服务器的主机(`HostName`)
可以看出，原始的`SSH URL`中,`User`和`HostName`都是一样的,导致本地计算机无法区分该采用哪个`ssh-key`去连接.<br>

但是通过修改`~/.ssh/config`文件,使用别名`Host`去映射`HostName`,再到`ssh key`,实现对两个账号的分离.
```c
通过如下方式来测试一下
$ ssh -T git@BooomYang
Hi BooomYang! You've successfully authenticated, but GitHub does not provide shell access.
$ ssh -T git@AllYang
Hi AllYang! You've successfully authenticated, but GitHub does not provide shell access.
```
此时两个账号已经分离开各司其职了.

###### 最后
虽然两个账号已经区分开,但是在执行`push/pull`操作的时候,并没有命令包含`Host`信息,此时系统依旧不知道该使用哪个账号,所以我们需要修改文件`yourRepo/.git/config`.
```c
$ vi /Users/yangwang/我的Demo/AllYangRepo/.git/config
执行命令可看到: 
[remote "origin"]
        url = git@github.com:AllYang/AllYangRepo.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```
发现url依旧是`git@gihtub`,`User`与`HostName`和之前是一样的问题.所以我们修改为
```c
url = git@AllYang:AllYang/AllYangRepo.git
```
同理，对于`AllYang/AllYangRepo`,我们修改url为
```c
url = git@BooomYang:BooomYang/BoomRepo.git
```
至此,解决了系统判断不了`ssh-key`导致的权限校验问题.



#### `HTTPS方案`
同理,我们要在代码仓库中能够区分采用哪个Github账号. <br>
* 在`BooomYang/BoomRepo`中 
```c
url=https://BooomYang@github.com/BooomYang/BoomRepo.git
```
* 在`AllYang/AllYangRepo`中
```c
https://AllYang@github.com/AllYang/AllYangRepo.git
```
原理就是将用户名添加到仓库的Git地址上,这样执行命令时系统就会用指定的用户名在`key-chain`中找到对应的认证信息,账号错乱的问题就解决了.
