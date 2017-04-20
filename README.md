# Multiple-account-authorization-processing-with-Git.

# 简单来讲,Git权限校验方式有两种,以github为例: 
 * 1).HTTPS URLs
 * 2).SSH URLs
 
 
一.HTTPS URLs
实际就是校验github输入的账户和密码. 

Mac系统中:keychain会存储用户第一次输入的账户密码,下次访问仓库时自动读取,不必重复输入.
#
# 非Mac:Git提供credential helper机制,下次访问时自动读取.
      1). $ git config --global credential.helper cache (输入的账号密码会在内存中缓存一段时间,默认15分钟)
      2). $ git config --global credential.helper store (输入的账号密码会存储在 ~/.git-credentials中)
      
二.SSH URLs 

ps.题外话
生成SSH key步骤: 
  1). # $ cd ~/.ssh
        $ ls
      查看是否有id_rsa、id_rsa.pub,如果有可跳过步骤2.
  
  2). # $ ssh-keygen -t rsa -C "your_email@example.com"  
      生成秘钥. 
      
      Generating public/private rsa key pair.
      # Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter] 
      这是提示生成路径. 默认按回车.
      
      Enter passphrase (empty for no passphrase): 
      # Enter same passphrase again:
      提示是否输入passphrase密码,如果输入则push时需要输入此密码. 
      
  3). 在github上添加ssh key,即id_rsa.pub的内容拷贝进去. 
  
  4). # $ ssh -T git@github.com
      测试该ssh. 
      若出现Hi username! You've successfully authenticated, but GitHub does not
      # provide shell access. 则成功. 
      
      
