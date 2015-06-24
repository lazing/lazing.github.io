---
layout: post
title:  "小措施增强Linux服务器安全"
date:   2012-08-06 12:06:41
categories: devops security ssh
---

黑客很多时候是个体力活。

挂着扫描器，漫无目的的寻找不设防的主机，植入后门，控制，卖给需要的人。

所以，一些基本的安全措施可以避免太过容易成为目标，下面就小小的介绍一些。

## 禁止root远程登录

作为默认系统管理账号root是最容易攻击的目标。禁止通过ssh远程登录是绝对必须的。

方法： 编辑 /etc/ssh/sshd_config

    PermitRootLogin no 
    

同时，请为管理员建立个人账户，并分配到sudoers用户组(默认为%admin)

    $ sudo adduser example_user 
    $ sudo usermod –a -G admin example_user 
    

## 修改SSHD默认端口

远程服务SSHD的默认端口22也是端口扫描的重点目标，修改为其他端口（通常为1024以上）可避免大部分攻击。

方法： 编辑 /etc/ssh/sshd_config

    port 8822 #default 22 
    

## 使用SCP代替FTP

FTP虽然方便，但是安全性一直被诟病。 后台文件管理时，用加密的SCP方式可以更好的解决这个问题。

SCP利用了SSHD的服务，所以不需要在服务器另外配置，直接调整账号权限即可。

Windows下可以使用软件winscp连接服务器。

官方网站： [http://winscp.net][1]

## 安装denyhosts

Denyhost可以帮你自动分析安全日志，直接禁止可疑主机暴力破解。 Debian用户可以直接使用apt安装

    $ sudo apt-get install denyhosts 
    

官方网站： <http://denyhosts.sourceforge.net/>

## 谨慎控制目录和文件权限，灵活使用用户组

例如，如果监控程序munin需要访问网站日志，请不要修改日志文件的权限设置，而是将munin加入www-data用户组

    $ sudo usermod -a -G www-data munin 
    

## 为系统程序使用专用账

尽量为每个系统程序使用专用账号，避免使用root 如mysql, munin 等，灵活使用 sudo -u example_user 等命令切换执行用户和用户组

## 从官方网站下载putty

Putty是非常流行的windows平台远程工具，但不要贪图方便随意下载。

如此重要且免费的软件，请从官方网站下载，并且最好进行完整性校验。 官方网站：

<http://www.chiark.greenend.org.uk/~sgtatham/putty/>

## 关闭不用的端口，以及设置iptables

利用 `sudo lsof -P | grep LISTEN` 即可检查服务器正在监听的端口和相关进程。 如果发现没有用的端口，可以配置相关软件设置监听范围。

一个更加全面的技巧是使用iptables过滤网络包。一个典型的配置如下：

    sudo apt-get install iptables  # 安装软件
    
    sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  # 允许所有已建立链接
    sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT  #允许ssh端口(22)
    sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT #允许80端口
    sudo iptables -A INPUT -j DROP # 阻止其他访问
    sudo iptables -I INPUT 1 -i lo -j ACCEPT  #允许本地环回 lo
    

利用 `sudo iptables -L -v` 可以检查当前设置

    Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
     pkts bytes target     prot opt in     out     source               destination         
        0     0 ACCEPT     all  --  lo     any     anywhere             anywhere
        0     0 ACCEPT     all  --  any    any     anywhere             anywhere            state RELATED,ESTABLISHED
        0     0 ACCEPT     tcp  --  any    any     anywhere             anywhere            tcp dpt:ssh
        0     0 ACCEPT     tcp  --  any    any     anywhere             anywhere            tcp dpt:www
        0     0 DROP       all  --  any    any     anywhere             anywhere
    

启用相关日志

    sudo iptables -I INPUT 5 -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
    

如果现在重启，所有配置将会丢失，可以安装`iptables-persistent`，自动导出和加载配置

    sudo apt-get install iptables-persistent
    

更多内容可参考 https://help.ubuntu.com/community/IptablesHowTo

希望这些有助于提高您网站的安全性

## 更新记录

2013\.11.02 增加iptables

 [1]: http://winscp.net/