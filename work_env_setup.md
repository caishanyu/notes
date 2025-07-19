# 工作环境配置 | working environment setup

记录一些办公环境配置

## 虚拟机和宿主机联网 | Networking between virtual machines and the host machine

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

