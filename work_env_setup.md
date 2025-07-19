# 工作环境配置

记录一些办公环境配置

## 虚拟机和宿主机联网

在Windows上运行Vmware虚拟机，虚拟机上运行Ubuntu系统，通过桥接的方式实现联网

> 所谓桥接模式，也就是虚拟机与宿主机处于同一个网段， 宿主机与虚拟机是平级关系， 例如两者都处于同一个局域网，具有相同的网关， 此时虚拟机相当于一台真实的设备。桥接模式下物理机上的vmnet0是物理机上的一个虚拟网卡，被windows隐藏了，无法看到， 用于在桥接情况下虚拟机上的网卡与物理机相连。

![bridge](https://github.com/caishanyu/notes/blob/main/images/bridgeNetwork.png)

以Vmware Work Station为例，在如下两个位置设置，这里我桥接的是wifi网卡

![1](https://github.com/caishanyu/notes/blob/main/images/virtualNetworkEditor.png)

![2](https://github.com/caishanyu/notes/blob/main/images/virtualMachineSetting.png)

设置完毕后查看两个设备的IP，可以看到在同一个网段内，尝试ping，没问题

![PCIP](https://github.com/caishanyu/notes/blob/main/images/pcIp.png)

![VMIP](https://github.com/caishanyu/notes/blob/main/images/ubuntuIp.png)

![ping](https://github.com/caishanyu/notes/blob/main/images/ping.png)

Ubuntu默认通过DHCP获取IP，我们可以将其设置为静态IP，避免后期IP更改

![staticIP](https://github.com/caishanyu/notes/blob/main/images/static.png)

## SSH配置

在工作中，经常在windows主机上通过SSH连接到虚拟机，像vscode，sourceInsight等工作软件就不用到虚拟机中下载使用，还可以使用mobaxterm终端工具登录虚拟机使用

### Ubuntu配置

Ubuntu需要开启SSH服务，作为SSH服务器

1. 安装SSH服务：`sudo apt-get  install openssh-server`
2. 启用SSH服务：`sudo /etc/init.d/ssh start`
3. 验证SSH服务已开启：`sudo service ssh status`，看到状态为`active`即可
4. 允许root用户登录ssh：
   1. 修改配置文件：`sudo vim /etc/ssh/sshd_config`，文件末尾添加`PermitRootLogin yes`
   2. 重启SSH服务，使修改生效：`sudo service ssh restart`

### windows宿主机设置

确保开启SSH服务即可，连接失败的话，检查两侧防火墙，TCP/UDP端口22开放情况

## Samba设置

使用Samba可以很方便的实现宿主机和虚拟机的文件互传，我们可以将虚拟机的文件路径映射到宿主机的磁盘中，在宿主机上就可以在这个路径操作虚拟机的文件

### Ubuntu配置

1. 安装samba：`sudo apt install samba`
2. 创建目录，用于共享，比如在用户目录下创建：`mkdir ~/share`；给共享目录足够的权限：`chmod -R 777 ~/share`
3. 修改samba配置，可以先备份原有配置：`cp /etc/samba/smb.conf ~/`；接着修改配置文件：`sudo vim /etc/samba/smb.conf`，在文件末尾添加配置，例如：

```
[share]
        comment = My Share
        path = /home/cai/share  # 共享目录路径
        browseable = yes
        writable = yes          # 是否可写
```

4. 设置samba用户名和密码：`sudo smbpasswd -a <name>`，`<name>`替换为自己想设置的用户名，按提示输入密码即可
5. 启用samba服务：`sudo systemctl start smbd.service`；验证服务是否启用：`sudo systemctl status smbd.service`
6. 设置开机自启动：`sudo systemctl enable smbd.service`

### windows宿主机配置

windows一般是默认开启SMB服务的

打开文件资源管理器，输入路径：`\\unbuntuIP`，比如我的是`\\192.168.0.106`，输入上边步骤4设置的用户名和密码，即可看到上边设置的共享目录

可以选择将其映射到网络驱动，方便后续访问，这里不赘述

