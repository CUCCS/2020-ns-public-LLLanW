# 常见蜜罐体验和探索

## 实验目的

* 了解蜜罐的分类和基本原理
* 了解不同类型蜜罐的适用场合
* 掌握常见蜜罐的搭建和使用

## 实验环境

* kali1(Attacker)：10.0.2.5
* kali2(Victim)：10.0.2.15
* 两台主机可互通

![pingcheck](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/pingcheck.png)  

## 实验过程

### 低交互蜜罐——ssh-Honeypot

* 选择原因：该蜜罐为一种极低交互式的简易蜜罐，配置docker容器即可观察。

#### 搭建蜜罐ssh-Honeypot过程

##### 安装docker-ce  

```1
#添加dockerce源
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
#添加密钥
curl -fsSL https://download.daocloud.io/docker/linux/ubuntu/gpg | sudo apt-key add -
```

![install](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/install.png)  

```2
#安装docker-ce
apt-get install docker-ce
#启动服务
systemctl start docker
#测试是否安装成功
sudo docker run hello-world
```

![check](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/checkdocker.png)  

##### 安装ssh

```3
make
ssh-keygen -t rsa -f ./ssh-honeypot.rsa
#在设定密码时，先将密码设定为空
```

![ssh](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/installssh.png)  

##### 安装ssh-Honeyport及其依赖

```4
apt install libssh-dev libjson-c-dev
```

![installapply](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/installapply.png)  

```5
#克隆仓库到本地
git clone https://github.com/random-robbie/docker-ssh-honey
#进入到仓库目录
cd docker-ssh-honey
#构建镜像
docker build . -t local:ssh-honeypot
#运行镜像
docker run -p 2234:22 local:ssh-honeypot
```

![installhoneypot](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/installhoneypot.png)  

![runhoneypot](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/intodocker.png)  

##### 进入容器并查看日志

```6
#查看该容器所对应的id
docker ps
#进入容器
docker exec -i -t id bash
#查看日志
tail -F ssh-honeypot.log
```

![into](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/intodocker1.png)  

![checklog](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/checklog.png)  

#### 攻击者主机对靶机进行访问

##### ssh远程登录

```7
#在攻击者主机上输入（需确保攻击者主机上已安装ssh）
ssh root@10.0.2.15 -p 2234
```

![trytossh](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/trytossh.png)  

在靶机蜜罐日志中**发现登录信息**，记录了**攻击者的ip**，**登录身份**，及**登录密码**  

##### nmap扫描

```8
#在攻击者主机上依次输入（需确保攻击者主机上已安装nmap）
#TCP connect scan
nmap -sT -P 2234 -n -vv 10.0.2.15
#TCP NULL scans
nmap -sN -P 2234 -n -vv 10.0.2.15
#TCP SYN scan
nmap -sS -P 2234 -n -vv 10.0.2.15
#TCP ping扫描
nmap -sP -P 2234 -n -vv 10.0.2.15
#TCP FIN scans
nmap -sF -P 2234 -n -vv 10.0.2.15
#TCP Xmas scans
nmap -sX -P 2234 -n -vv 10.0.2.15
```

![nmapt](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapt.png)  

![nmapn](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapn.png)  

![nmaps](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmaps.png)  

![nmapp](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapp.png)  

![nmapf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapf.png)  

![nmapx](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapx.png)  

六种nmap扫描方法均在蜜罐日志中**没有记录**

### 中交互蜜罐——Cowrie

* 选择原因：该蜜罐较其他蜜罐在GitHub保持更新

#### 搭建蜜罐Cowrie过程

##### 安装Cowrie并运行

```9
docker pull cowrie/cowrie
docker run -p 2222:2222 cowrie/cowrie
```

![pullcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/pullcowrie.png)  

##### 与低交互蜜罐相似，进入容器并查看日志

```10
#查看该容器所对应的id
docker ps
#进入容器
docker exec -i -t id bash
#查看日志
tail -F ssh-honeypot.log
```

#### 攻击者主机对靶机进行ssh及nmap扫描

##### ssh

```7
#在攻击者主机上输入（需确保攻击者主机上已安装ssh）
ssh root@10.0.2.15 -p 2234
```

![sshcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/sshcowrie.png)  

如图，攻击者远程ssh**可以被靶机蜜罐日志记录**

##### nmap

```8
#在攻击者主机上依次输入（需确保攻击者主机上已安装nmap）
#TCP connect scan
nmap -sT -P 2234 -n -vv 10.0.2.15
#TCP NULL scans
nmap -sN -P 2234 -n -vv 10.0.2.15
#TCP SYN scan
nmap -sS -P 2234 -n -vv 10.0.2.15
#TCP ping扫描
nmap -sP -P 2234 -n -vv 10.0.2.15
#TCP FIN scans
nmap -sF -P 2234 -n -vv 10.0.2.15
#TCP Xmas scans
nmap -sX -P 2234 -n -vv 10.0.2.15
```

六种nmap扫描方法中，**仅`TCP connect scan`在靶机蜜罐中获得记录**
![nmapcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmaptcowries.png)  

![nmapcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapncowrie.png)  

![nmapcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapscowrie.png)  

![nmapcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmappcowrie.png)  

![nmapcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapfcowrie.png)  

![nmapcowrie](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp11/img/nmapcowries.png)  

## 参考资料

[师姐的实验报告](https://github.com/CUCCS/2019-NS-Public-chencwx/tree/ns_chap0x11/ns_chapter11)  
[ssh-honeypot](https://github.com/droberson/ssh-honeypot)  
[cowrie](https://github.com/cowrie/cowrie)  
[cowrie readthedocs](https://cowrie.readthedocs.io/en/latest/index.html)  
