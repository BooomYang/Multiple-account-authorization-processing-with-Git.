
### 简单来讲,Git权限校验方式有两种,以Github为例: 
    
#### `HTTPS URLs`<br>
实际就是校验Github输入的账户和密码.
    
Mac系统中:keychain会存储用户第一次输入的账户密码,下次访问仓库时自动读取,不必重复输入.<br>
非Mac:Git提供credential helper机制,下次访问时自动读取.
```c
$ git config --global credential.helper cache (输入的账号密码会在内存中缓存一段时间,默认15分钟)
$ git config --global credential.helper store (输入的账号密码会存储在 ~/.git-credentials中)   
```
#### `SSH URLs`
实际就是校验本地存储的私钥和添加至Github账户的公钥.     

在使用SSH校验之前,需要现在本地计算机中生成`SSH Keypair`.

##### ps.题外话
生成SSH key步骤: 
```c
  1).  $ cd ~/.ssh  
       $ ls
       查看是否有`id_rsa`、`id_rsa.pub`,如果有可跳过步骤2.

  2). $ ssh-keygen -t rsa -C "your_email@example.com"  
      生成秘钥对. 
      
      Generating public/private rsa key pair.
      Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter] 
      这是提示生成路径. 默认按回车.
      
      Enter passphrase (empty for no passphrase): 
      Enter same passphrase again:
      提示是否输入passphrase密码,如果输入则push时需要输入此密码. 
      
      秘钥对创建好后,秘钥存储在本地(`~/.ssh/id_rsa`),公钥(`~/.ssh/id_rsa.pub`)应该添加到Github的账户.
      
  3). 在github上添加ssh key,即id_rsa.pub的内容拷贝进去. 
  
  4). # $ ssh -T git@github.com
      测试该ssh. 
      若出现Hi username! You've successfully authenticated, but GitHub does not
      # provide shell access. 则成功. 
 ```
      
      
