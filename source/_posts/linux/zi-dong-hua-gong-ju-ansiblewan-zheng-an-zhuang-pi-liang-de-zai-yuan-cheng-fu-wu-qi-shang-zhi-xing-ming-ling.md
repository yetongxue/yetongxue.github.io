---
title: 自动化工具 ansible完整安装
date: 2016-11-16 00:00:00
tags: [Linux]
categories: 
  - Linux
---

批量的在远程服务器上执行命令

# 1.什么是ansible

ansible是个什么东西呢？官方的title是“Ansible is Simple IT Automation”——简单的自动化IT工具。这个工具的目标有这么几项：让我们自动化部署APP；自动化管理配置项；自动化的持续交付；自动化的（AWS）云服务管理。

<!--more-->

所有的这几个目标本质上来说都是在一个台或者几台服务器上，执行一系列的命令而已。就像我之前有介绍过的Fabric，以及我们基于Fabric开发的自动化应用部署的工具： Essay 。都是做了这么个事——批量的在远程服务器上执行命令 。

那么fabric和ansible有什么差别呢？简单来说fabric像是一个工具箱，提供了很多好用的工具，用来在Remote执行命令，而Ansible则是提供了一套简单的流程，你要按照它的流程来做，就能轻松完成任务。这就像是库和框架的关系一样。

当然，它们之间也是有共同点的——都是基于 paramiko 开发的。这个paramiko是什么呢？它是一个纯Python实现的ssh协议库。因此fabric和ansible还有一个共同点就是不需要在远程主机上安装client/agents，因为它们是基于ssh来和远程主机通讯的。

# 2.快速安装

上面简单介绍了下这是个什么东西，怎么安装呢？也很简单，因为ansible是python开发的，因此可以这么安装:

使用yum安装

```
yum install epel-release -y  
yum install ansible -y  

```

# 3.配置

ansible通过文件来定义你要管理的主机,也就是说把你需要的管理的主机ip写到一个文件中即可。
这个文件一般名为hosts，它可以放在多个路径下，也可以自定义名称和路径。 默认我们用/etc/ansible/hosts这个文件即可

hosts文件的格式为：

```
[node]
10.2.2.121  
10.2.2.122  

```

默认ssh端口是22，如果主机端口号是其他的，在ip后加:端口号即可，如10.2.1.203的端口是2211，属于test组，格式如下：

```
[test]
10.2.1.201  
10.2.1.202  
10.2.1.203:2211  

```

# 4.使用ssh-keygen产生ssh密钥

```
[root@test-201 ~]# ssh-keygen 
Generating public/private rsa key pair.  
Enter file in which to save the key (/root/.ssh/id_rsa):  
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in /root/.ssh/id_rsa.  
Your public key has been saved in /root/.ssh/id_rsa.pub.  
The key fingerprint is:  
dd:20:23:7c:1a:2e:01:bf:b1:67:7a:08:87:5f:e6:7e root@test-201  
The key’s randomart image is:  
+–[ RSA 2048]—-+ 
| . | 
| o . | 
| + + + . | 
| . * = + o | 
| o = B S . . | 
| + X | 
| + o | 
| o E | 
| .. | 
+—————–+

```

将公钥发送到要管理的服务器 使用ssh-copy-id命令

比如要发送到10.2.31.202，使用如下命令：

```
[root@test-201 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub 10.2.31.202 
root@10.2.31.202’s password:  
Now try logging into the machine, with “ssh ‘10.2.31.202’”, and check in:

.ssh/authorized_keys

to make sure we haven’t added extra keys that you weren’t expecting.  

```

# 5.使用ansible

命令格式如下： `ansible + 主机组名称 + -m + 模块名称 + -a + 参数`

主机组名称，即hosts中定义的主机组名称

-m 指使用模块，后加指定的模块名称  
-a 指传给模块的参数

在不指定模块时，默认调用command模块。

如我们想看下test组上的服务器的/tmp下面有哪些文件，可以使用如下命令

`ansible test -a “ls /tmp”`

```
[root@test-201 ~]# ansible test -a "ls /tmp"
10.2.31.203 | SUCCESS | rc=0 >>  
ansible_EMEGZI  
testabcdefg  

```

我们可以使用copy模块，将本地文件发送到目标服务器上，如：

`ansible test -m copy -a “src=/root/install.log dest=/tmp”`

这个命令是将本地的/root/install.log发送到test组的/tmp下，执行的效果如下：

```
[root@test-201 ~]# ansible test -m copy -a "src=/root/install.log dest=/tmp"
10.2.31.203 | SUCCESS => {  
    "changed": true, 
    "checksum": "114ee153946d9cd2e690c405e5796a4fcc400542", 
    "dest": "/tmp/install.log", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "17b18780156a31a65d62a324110d686e", 
    "mode": "0644", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:admin_home_t:s0", 
    "size": 43688, 
    "src": "/root/.ansible/tmp/ansible-tmp-1466157410.68-191787255687953/source", 
    "state": "file", 
    "uid": 0
}

```

你可以使用ansible-doc –list查看当前的所有模块

```
[root@test-201 ~]# ansible-doc  --list                       
….
….                                                        
authorized_key                     Adds or removes an SSH authorized key  
azure                              create or terminate a virtual machine in azure  
azure_rm_deployment                Create or destroy Azure Resource Manager template deployments  
azure_rm_networkinterface          Manage Azure network interfaces.  
azure_rm_networkinterface_facts    Get network interface facts.  
azure_rm_publicipaddress           Manage Azure Public IP Addresses.  
azure_rm_publicipaddress_facts     Get public IP facts.  
azure_rm_resourcegroup             Manage Azure resource groups.  
azure_rm_resourcegroup_facts       Get resource group facts.  
azure_rm_securitygroup             Manage Azure network security groups.  
….
…

```

ansible自带了很多丰富的模块，详细请看：
[http://docs.ansible.com/ansible/list_of_all_modules.html](http://docs.ansible.com/ansible/list_of_all_modules.html)

# 6.小技巧:

有时候如果想直接操作某台服务器,但又没有在hosts里定义这台服务器时,可以使用如下命令:

`ansible all -i ‘服务器ip,’`  
注意服务器ip后面要加个,

如 `ansible all -i ‘10.2.31.201,’ -u test -k -a ‘uptime’`

BUG?

目前遇到两个bug(也可能是我的使用方式不对,正在关注中) 1.在中文路径下无法使用.  
如果在一个含中文的路径下面使用ansible,会无法执行,提示:   UnicodeDecodeError: ‘ascii’ codec can’t decode byte 0xe4 in position 14  
所以不要跑到中文路径下面去执行ansible

2.su命令不能用.
使用su命令不成功,无在目标机器上通过一个普通用户su切换为root执行相关命令 错误如下: 
```
ansible Timeout (12s) waiting for privilege escalation prompt
```