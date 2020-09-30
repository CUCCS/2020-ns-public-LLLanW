# 基于 VirtualBox 的网络攻防基础环境搭建

## 实验目的

* 掌握 VirtualBox 虚拟机的安装与使用；
* 掌握 VirtualBox 的虚拟网络类型和按需配置；
* 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境

* VirtualBox 虚拟机
* 攻击者主机（Attacker）：Kali Rolling 2109.2
* 网关（Gateway, GW）：Debian Buster
* 靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

## 实验要求

### 虚拟硬盘配置成多重加载

* 注：由于创建多重加载的时间与后面进行实验相隔太久，部分虚拟机名字有改变，最终名字见“各个虚拟机对应的ip地址”一栏。

![多重加载1](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/%E5%A4%9A%E9%87%8D%E5%8A%A0%E8%BD%BD1.jpg)

![多重加载2](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/%E5%A4%9A%E9%87%8D%E5%8A%A0%E8%BD%BD2.jpg)

![多重加载3](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/%E5%A4%9A%E9%87%8D%E5%8A%A0%E8%BD%BD3.jpg)

### 搭建满足如下拓扑图所示的虚拟机网络拓扑

#### 各个虚拟机的网络配置

**准备工作：** 由于网关和攻击者需要用到NAT网络，故在 管理->全局设定 中先进行如下设置

![prepare](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/prepare2.png)

##### Gateway

![Gateway](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Gatenet.png)

###### 修改网络配置文件

`vi /etc/network/interfaces`，将该链接中*etc-network-interfaces*部分粘贴到文件中：https://gist.github.com/c4pr1c3/8d1a4550aa550fabcbfb33fad9718db1
修改完成，执行`systemctl restart networking`,重启网络服务

###### 配置两块内部网卡ip地址

此时**两块内部网卡仍未显示ip地址，需要手动开启**
执行`/sbin/ifup enp0s9`,`/sbin/ifup enp0s10`即可完成

###### 安装并配置dnsmasq
`apt update && apt install dnsmasq`
修改配置文件 `vi /etc/dnsmasq.conf`,将该链接中*diff dnsmasq.conf dnsmasq.conf.bak*部分中内容，对应文件中指定内容进行修改：https://gist.github.com/c4pr1c3/8d1a4550aa550fabcbfb33fad9718db1
新建两个文件：`vi /etc/dnsmasq.d/gw-enp010.conf`,`vi /etc/dnsmasq.d/gw-enp09.conf`并将该链接中余下两个部分分别写入：https://gist.github.com/c4pr1c3/8d1a4550aa550fabcbfb33fad9718db1

##### Kali-attacker

![attacker](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/attackernet.png)

##### Victim-XP-1

![XP1](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/XP1net.png)

* 配置XP系统网络
**在虚拟机外部选择网卡时，注意在设置->网络->网卡1->高级->控制芯片中选择“PCnet-FAST III”网络，否则无法正常建立连接**
在“开始->控制面板->网络和Internet连接->网络连接->(右键本地连接)属性->(双击)Internet”协议中，手动修改ip地址、掩码及网关
**注意，在配置完成后关闭网络防火墙，否则无法ping通**

##### Victim-Kali-1

![Kali1](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/kalinet.png)

##### Victim-XP-2

![XP2](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/XP2net.png)
操作同Victim-XP-1

##### Victim-Debian-2

![debian2](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Denet.png)

#### 各个虚拟机对应的ip地址

* Gateway：10.0.2.4
* Kali-attacker：10.0.2.5
* Victim-XP-1：172.16.111.101
* Victim-Kali-1：172.16.111.114
* Victim-XP-2：172.16.222.101
* Victim-Debian-2：172.16.222.116

### 完成以下网络连通性测试

* 靶机可以直接访问攻击者主机[√]
![Vping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Vping1.png)
![Vping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Vping2.png)
![Vping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Vping3.png)
![Vping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Vping4.png)

* 攻击者主机无法直接访问靶机[√]
![Aping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/attackerCantping.jpg)

* 网关可以直接访问攻击者主机和靶机[√]
![Gping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Gping1.png)
![Gping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Gping2.png)
![Gping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Gping3.png)
![Gping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Gping4.png)
![Gping](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/Gping5.png)

* 靶机的所有对外上下行流量必须经过网关[√]
![log](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/log.png)

* 所有节点均可以访问互联网[√]
![surf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/surf1.png)
![surf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/surf2.png)
![surf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/surf3.png)
![surf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/surf4.png)
![surf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/surf5.png)
![surf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp01/img/surf6.png)
