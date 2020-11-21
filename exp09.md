# Snort

## 实验目的

进行入侵检测的软件实现体验，以工具`snort`为主

## 实验要求

通过配置`snort`不同规则来进行网络入侵检测

## 实验环境

* 两台debian主机，可互通
* 工具：snort、guardian、nmap

## 实验过程

### 实验一 配置snort为嗅探模式

#### 安装snort

```1
# 禁止在apt安装时弹出交互式配置界面
export DEBIAN_FRONTEND=noninteractive
apt install snort
```

![install](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/install_snort.png)  

#### 配置过程及结果

```2
# 显示IP/TCP/UDP/ICMP头
snort –v
```

![snortv](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/snortv.png)  

```3
# 显示应用层数据
snort -vd
```

![snortvd](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/snortvd.png)  

```4
# 显示数据链路层报文头
snort -vde
```

![snortvde](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/snortvde.png)  

```5
# -b 参数表示报文存储格式为 tcpdump 格式文件
# -q 静默操作，不显示版本欢迎信息和初始化信息
snort -q -v -b -i eth1 "port not 22"

# 使用 CTRL-C 退出嗅探模式
# 嗅探到的数据包会保存在 /var/log/snort/snort.log.<epoch timestamp>
# 其中<epoch timestamp>为抓包开始时间的UNIX Epoch Time格式串
# 可以通过命令 date -d @<epoch timestamp> 转换时间为人类可读格式
# exampel: date -d @1511870195 转换时间为人类可读格式
# 上述命令用tshark等价实现如下：
tshark -i eth1 -f "port not 22" -w 1_tshark.pcap
```

![snort5](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/snort5.png)  

### 实验二 配置并启用snort内置规则

```7
# /etc/snort/snort.conf 中的 HOME_NET 和 EXTERNAL_NET 需要正确定义
# 例如，学习实验目的，可以将上述两个变量值均设置为 any
snort -q -A console -b -i eth1 -c /etc/snort/snort.conf -l /var/log/snort/
```

![snortconf](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/snortconf.png)  

#### 启用内置规则

`snort -q -A console -b -i enp0s3 -c /etc/snort/snort.conf -l /var/log/snort/`

![opensnortrunle](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/opensnortrunle.png)  

### 实验三 自定义snort规则

#### 添加规则

```8
# 新建自定义 snort 规则文件
cat << EOF > /etc/snort/rules/cnss.rules
alert tcp \$EXTERNAL_NET any -> \$HTTP_SERVERS 80 (msg:"Access Violation has been detected on /etc/passwd ";flags: A+; content:"/etc/passwd"; nocase;sid:1000001; rev:1;)
alert tcp \$EXTERNAL_NET any -> \$HTTP_SERVERS 80 (msg:"Possible too many connections toward my http server"; threshold:type threshold, track by_src, count 100, seconds 2; classtype:attempted-dos; sid:1000002; rev:1;)
EOF
```

![rulesadd](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/rulesadd.png)  

```9
# 添加配置代码到 /etc/snort/snort.conf
include $RULE_PATH/cnss.rules
```

![rulesadd](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/snortaddconfig.png)  

#### 启动

`snort -q -A console -b -i enp0s3 -c /etc/snort/snort.conf -l /var/log/snort/`

![opensnortrunle](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/opensnortrunle.png)  

### 实验四 和防火墙联动

#### snort所在的主机下载guardian，并运行snort

```10
# 下载压缩包
wget https://c4pr1c3.gitee.io/cuc-ns/chap0x09/attach/attach/guardian.tar.gz

# 解压缩 Guardian-1.7.tar.gz
tar zxf guardian.tar.gz

# 安装 Guardian 的依赖 lib
apt install libperl4-corelibs-perl

# 开启 snort
snort -q -A fast -b -i eth1 -c /etc/snort/snort.conf -l /var/log/snort/
```

![download](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/downloadG.png)  

#### 配置guardian并运行

```11
# 编辑 guardian.conf 并保存，确认以下2个参数的配置符合主机的实际环境参数。

HostIpAddr      10.0.2.15
Interface       enp0s3
```

![guarconfig](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/guarconfig.png)  

```12
# 启动 guardian.pl
perl guardian.pl -c guardian.conf
```

![runguar](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/runguar.png)  

#### 另一台主机用nmap扫描

```13
nmap 10.0.2.15 -A -T4 -n -vv
```

![nmap](https://github.com/CUCCS/2020-ns-public-LLLanW/blob/exp09/img/nmap.png)  

```14
guardian.conf 中默认的来源IP被屏蔽时间是 60 秒（屏蔽期间如果黑名单上的来源IP再次触发snort报警消息，则屏蔽时间会继续累加60秒）

root@KaliRolling:~/guardian# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
REJECT     tcp  --  10.0.2.6       0.0.0.0/0            reject-with tcp-reset
DROP       all  --  10.0.2.6       0.0.0.0/0

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

# 1分钟后，guardian.pl 会删除刚才添加的2条 iptables 规则
root@KaliRolling:~/guardian# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

## 遇到的问题及其解决办法

* Q:安装snort出错
A:选择不进行交互时，默认选择监听eth0网络，网段192.168.0/16， 而本机网络名称enp0s3，网段10.0.2.15/24，不一致造成错误
**解决：手动修改网段及网络名称**

* Q:安装完成snort，service启动但无法使用
A:su切换到root用户时，环境变量仍为普通用户的变量，故需找到snort可执行程序的位置并使用
**解决：su-切换到root用户**

## 参考资料

https://github.com/CUCCS/2019-NS-Public-chencwx