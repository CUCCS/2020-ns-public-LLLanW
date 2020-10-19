# 基于Scapy编写端口扫描器

## 实验目的

掌握网络扫描之端口状态探测的基本原理

## 实验环境

* python + scapy

### 实验所用虚拟机及其ip

* Gateway：10.0.2.4
* Kali-attacker：172.16.111.143
* Victim-Kali-1：172.16.111.114

## 实验要求

### 预期结果

* 开放状态：攻击者向靶机发送SYN包，能完成三次握手，收到ACK  
* 关闭状态：只收到一个RST包  
`sudo ufw disable`  
`systemctl stop apache2(用于关闭80端口)`  
`systemctl stop dnsmasq(用于关闭53端口)`  
* 端口过滤状态：什么都没收到  
靶机添加规则过滤80端口`sudo ufw enable && sudo ufw deny 80/tcp`

### 完成以下扫描技术的编程实现

#### TCP connect scan

![connect](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/connection.png)  

>from scapy.all import *
def tcpconnect(dst_ip,dst_port,timeout=10):
    pkts=sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="S"),timeout=timeout)
    if (pkts is None):
        print("FILTER")
    elif(pkts.haslayer(TCP)):
        if(pkts[1].flags=='AS'):
            print("OPEN")
        elif(pkts[1].flags=='AR'):
                print("CLOSE")
tcpconnect('172.16.111.114',80)

* 开放状态  
![open](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/connection-open.png)  
* 关闭状态  
![close](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/connection-close.png)  
* 过滤状态  
![filter](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/connection-filter.png)  

#### TCP stealth scan

![stealth](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/STEALTH.png)  

>from scapy.all import *
def tcpstealthscan(dst_ip , dst_port , timeout = 10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="S"),timeout=10)
    if (pkts is None):
        print ("Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x12):
            send_rst = sr(IP(dst=dst_ip)/TCP(dport=dst_port,flags="R"),timeout=10)
            print ("Open")
        elif (pkts.getlayer(TCP).flags == 0x14):
            print ("Closed")
        elif(pkts.haslayer(ICMP)):
            if(int(pkts.getlayer(ICMP).type)==3 and int(stealth_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
                print ("Filtered")
tcpstealthscan('172.16.111.114',80)

* 开放状态  
![open](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/stealth-open.png)  
* 关闭状态  
![close](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/stealth-close.png)  
* 过滤状态  
![filter](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/stealth-filter.png)  

#### TCP Xmas scan

![xmas](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/XMAS.png)  

>from scapy.all import *
def Xmasscan(dst_ip , dst_port , timeout = 10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="FPU"),timeout=10)
    if (pkts is None):
        print ("Open|Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x14):
            print ("Closed")
    elif(pkts.haslayer(ICMP)):
        if(int(pkts.getlayer(ICMP).type)==3 and int(pkts.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print ("Filtered")
Xmasscan('172.16.111.114',80)

* 开放状态  
![open](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/xmas-open.png)  
* 关闭状态  
![close](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/xmas-close.png)  
* 过滤状态  
![filter](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/xmas-filter.png)  

#### TCP fin scan

![fin](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/FIN.png)  

>from scapy.all import *
def finscan(dst_ip , dst_port , timeout = 10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="F"),timeout=10)#发送FIN包
    if (pkts is None):
        print ("Open|Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x14):
            print ("Closed")
    elif(pkts.haslayer(ICMP)):
        if(int(pkts.getlayer(ICMP).type)==3 and int(pkts.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print ("Filtered")
finscan('172.16.111.114',80)

* 开放状态  
![open](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/fin-open.png)  
* 关闭状态  
![close](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/fin-close.png)  
* 过滤状态  
(与开放状态结果相同)

#### TCP null scan

![null](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/NULL.png)  

>from scapy.all import *
def nullscan(dst_ip , dst_port , timeout = 10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags=""),timeout=10)
    if (pkts is None):
        print ("Open|Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x14):
            print ("Closed")
    elif(pkts.haslayer(ICMP)):
        if(int(pkts.getlayer(ICMP).type)==3 and int(pkts.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print ("Filtered")
nullscan('172.16.111.114',80)

* 开放状态(与TCP fin扫描结果相同)  
* 关闭状态(与TCP fin扫描结果相同)  
* 过滤状态(与TCP fin扫描结果相同)  

#### UDP scan

![UDP](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp05/img/UDP.png)  

>from scapy.all import *
def udpscan(dst_ip,dst_port,dst_timeout = 10):
    resp = sr1(IP(dst=dst_ip)/UDP(dport=dst_port),timeout=dst_timeout)
    if (resp is None):
        print("Open|Filtered")
    elif (resp.haslayer(UDP)):
        print("Open")
    elif(resp.haslayer(ICMP)):
        if(int(resp.getlayer(ICMP).type) == 3 and int(resp.getlayer(ICMP).code) == 3):
            print("Closed")
        elif(int(resp.getlayer(ICMP).type) == 3 and int(resp.getlayer(ICMP).code) in [1,2,9,10,13]):
            print("Filtered")
        elif(resp.haslayer(IP) and resp.getlayer(IP).proto==IP_PROTOS.udp):
            print("Open")
udpscan('172.16.111.114',53)

* 开放状态(与TCP fin扫描结果相同)  
* 关闭状态(与TCP fin扫描结果相同)  
* 过滤状态(与TCP fin扫描结果相同)  

## 参考资料

https://github.com/CUCCS/2019-NS-Public-chencwx
