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

### 宿主机设置

确保开启SSH服务即可，连接失败的话，检查两侧防火墙，TCP/UDP端口22开放情况

