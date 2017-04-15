---
title: LVS的3种负载均衡技术的测试
date: 2016-03-22 22:58:37
categories: [LVS]
tags: [LVS, 集群, 技术人生, NAT, TUN, DR]
---
在测试之前安装ipvsadm
安装之前需要安装依赖：

```
yum install kernel-devel gcc openssl openssl-devel popt
```

到官网 【http://www.linuxvirtualserver.org 】下载ipvsadm时需要注意查看服务器内核版本

查询你的服务器内核版本：
```
rpm -q kernel-devel
```

根据服务器内核版本选择ipvsadm
下载后安装

```
tar -xvf ipvsadm-x.xx.tar.gz

cd ipvsadm-x.xx

make && make install
```

测试用的是3台CentOS7主机
node1    负载均衡服务器
node2    主机1
node3    主机2

**第一种负载均衡：NAT**
根据下图配置
![NAT搭建图](https://1csh1.github.io/img/LVS的3种负载均衡技术的测试/NAT搭建图.jpg)

在node1中创建脚本：lvs-dr-node1.sh

```
#!/bin/sh
#------mini-HOWTO-setup-LVS-NAT-director----------
 
#set ip_forward ON for vs-nat director (1 on, 0 off).
cat /proc/sys/net/ipv4/ip_forward
echo "1" >/proc/sys/net/ipv4/ip_forward
 
#director is gw for realservers
#turn OFF icmp redirects (1 on, 0 off)
echo "0" >/proc/sys/net/ipv4/conf/all/send_redirects
cat       /proc/sys/net/ipv4/conf/all/send_redirects
echo "0" >/proc/sys/net/ipv4/conf/default/send_redirects
cat       /proc/sys/net/ipv4/conf/default/send_redirects
echo "0" >/proc/sys/net/ipv4/conf/eno16777736/send_redirects
cat       /proc/sys/net/ipv4/conf/eno16777736/send_redirects
 
#setup DIP
/sbin/ifconfig eno16777736 10.0.0.1 broadcast 10.0.0.255 netmask 255.255.255.0
 
#setup VIP
/sbin/ifconfig eno16777736:0 192.168.1.11 broadcast 192.168.1.255 netmask 255.255.255.0
 
#set default gateway
/sbin/route add default gw 192.168.1.1 netmask 0.0.0.0 metric 1
 
#clear ipvsadm tables
/sbin/ipvsadm -C
 
#install LVS services with ipvsadm
#add telnet to VIP with rr sheduling
/sbin/ipvsadm -A -t 192.168.1.11:80 -s rr
 
#first realserver
#forward telnet to realserver 10.0.0.2 using LVS-NAT (-m), with weight=1
/sbin/ipvsadm -a -t 192.168.1.11:80 -r 10.0.0.2:80 -m -w 1
#check that realserver is reachable from director
ping -c 1 10.0.0.2
#second realserver
#forward telnet to realserver 10.0.0.3 using LVS-NAT (-m), with weight=1
/sbin/ipvsadm -a -t 192.168.1.11:80 -r 10.0.0.3:80 -m -w 1
#checking if realserver is reachable from director
ping -c 1 10.0.0.3
 
#list ipvsadm table
/sbin/ipvsadm
#------mini-HOWTO-setup-LVS-NAT-director----------
```

在node2中创建脚本：lvs-dr-node2.sh

```
#!/bin/sh
#---------mini-HOWTO-setup-LVS-NAT-realserver-------
#setup IP
/sbin/ifconfig eno16777736 10.0.0.2 broadcast 10.0.0.255 netmask 255.255.255.0
#installing default gw 10.0.0.1 for vs-nat'
/sbin/route add default gw 10.0.0.1
#show routing table
/bin/netstat -rn
 
#checking if DEFAULT_GW is reachable
ping -c 1 10.0.0.1
 
#looking for VIP on director from realserver
ping -c 1 10.0.0.3
 
#set_realserver_ip_forwarding to OFF (1 on, 0 off).
echo "0" >/proc/sys/net/ipv4/ip_forward
cat       /proc/sys/net/ipv4/ip_forward
#---------mini-HOWTO-setup-LVS-NAT-realserver-------
```

在node3中创建脚本：lvs-nat-node3.sh

```
#!/bin/sh
#---------mini-HOWTO-setup-LVS-NAT-realserver-------
#setup IP
/sbin/ifconfig eno16777736 10.0.0.3 broadcast 10.0.0.255 netmask 255.255.255.0
#installing default gw 10.0.0.1 for vs-nat'
/sbin/route add default gw 10.0.0.1
#show routing table
/bin/netstat -rn
 
#checking if DEFAULT_GW is reachable
ping -c 1 10.0.0.1
 
#looking for VIP on director from realserver
ping -c 1 10.0.0.2
 
#set_realserver_ip_forwarding to OFF (1 on, 0 off).
echo "0" >/proc/sys/net/ipv4/ip_forward
cat       /proc/sys/net/ipv4/ip_forward
#---------mini-HOWTO-setup-LVS-NAT-realserver-------
```

分别给脚本附加执行权限

```
chmod -x lvs-nat-xxxxx.sh
```

分别运行3个脚本

```
./lvs-nat-xxxx.sh
```

NAT测试结果
![NAT结果图](https://1csh1.github.io/img/LVS的3种负载均衡技术的测试/NAT结果图.jpg)

NAT脚本文件：
[NAT脚本文件](https://1csh1.github.io/file/LVS的3种负载均衡技术的测试/nat.rar)


**第二种负载均衡：TUN**
根据下图配置
![TUN搭建图](https://1csh1.github.io/img/LVS的3种负载均衡技术的测试/TUN搭建图.jpg)
在node1中创建脚本：lvs-tun-node1.sh

```
#!/bin/bash
#---------------mini-rc.lvs_dr-director------------------------
#set ip_forward OFF for lvs-dr director (1 on, 0 off)
#(there is no forwarding in the conventional sense for LVS-DR)
cat       /proc/sys/net/ipv4/ip_forward
echo "0" >/proc/sys/net/ipv4/ip_forward
 
#director is not gw for realservers: leave icmp redirects on
echo 'setting icmp redirects (1 on, 0 off) '
echo "1" >/proc/sys/net/ipv4/conf/all/send_redirects
cat       /proc/sys/net/ipv4/conf/all/send_redirects
echo "1" >/proc/sys/net/ipv4/conf/default/send_redirects
cat       /proc/sys/net/ipv4/conf/default/send_redirects
echo "1" >/proc/sys/net/ipv4/conf/eno16777736/send_redirects
cat       /proc/sys/net/ipv4/conf/eno16777736/send_redirects
 
#setup DIP
/sbin/ifconfig eno16777736 192.168.1.101 broadcast 192.168.1.255 netmask 255.255.255.0
 
#add ethernet device and routing for VIP 192.168.1.11
/sbin/ifconfig tunl0 192.168.1.11 broadcast 192.168.1.11 netmask 255.255.255.255
/sbin/route add -host 192.168.1.11 dev tunl0
#listing ifconfig info for VIP 192.168.1.11
/sbin/ifconfig tunl0
 
#check VIP 192.168.1.11 is reachable from self (director)
/bin/ping -c 1 192.168.1.11
#listing routing info for VIP 192.168.1.11
/bin/netstat -rn
 
#setup_ipvsadm_table
#clear ipvsadm table
/sbin/ipvsadm -C
#installing LVS services with ipvsadm
#add telnet to VIP with round robin scheduling
/sbin/ipvsadm -A -t 192.168.1.11:80 -s rr
 
#forward telnet to realserver using direct routing with weight 1
/sbin/ipvsadm -a -t 192.168.1.11:80 -r 192.168.1.102:80 -i -w 1
#check realserver reachable from director
ping -c 1 192.168.1.102
 
#forward telnet to realserver using direct routing with weight 1
/sbin/ipvsadm -a -t 192.168.1.11:80 -r 192.168.1.103:80 -i -w 1
#check realserver reachable from director
ping -c 1 192.168.1.103
 
#displaying ipvsadm settings
/sbin/ipvsadm
 
#not installing a default gw for LVS_TYPE vs-dr
#---------------mini-rc.lvs_dr-director------------------------
```

在node2中创建脚本：lvs-tun-node2.sh

```
#!/bin/bash
#----------mini-rc.lvs_dr-realserver------------------
#setup IP
/sbin/ifconfig eno16777736 192.168.1.103 broadcast 192.168.1.255 netmask 255.255.255.0
#installing default gw 192.168.1.1 for vs-dr
/sbin/route add default gw 192.168.1.1
#showing routing table
/bin/netstat -rn
#checking if DEFAULT_GW 192.168.1.1 is reachable
ping -c 1 192.168.1.1
 
#set_realserver_ip_forwarding to OFF (1 on, 0 off).
echo "0" >/proc/sys/net/ipv4/ip_forward
cat       /proc/sys/net/ipv4/ip_forward
 
#looking for DIP 192.168.1.101
ping -c 1 192.168.1.101
 
#looking for VIP (will be on director)
ping -c 1 192.168.1.11
 
#install_realserver_vip
/sbin/ifconfig tunl0 192.168.1.11 broadcast 192.168.1.11 netmask 0xffffffff up
#ifconfig output
/sbin/ifconfig tunl0
#installing route for VIP 192.168.1.11 on device tunl0
/sbin/route add -host 192.168.1.11 dev tunl0
#listing routing info for VIP 192.168.1.11
/bin/netstat -rn
 
#hiding interface tunl0, will not arp
echo "1" >/proc/sys/net/ipv4/conf/all/hidden
cat       /proc/sys/net/ipv4/conf/all/hidden
echo "1" >/proc/sys/net/ipv4/conf/tunl0/hidden
cat       /proc/sys/net/ipv4/conf/tunl0/hidden
 
#----------mini-rc.lvs_dr-realserver------------------
```

在node3中创建脚本：lvs-tun-node3.sh

```
#!/bin/bash
#----------mini-rc.lvs_dr-realserver------------------
#setup IP
/sbin/ifconfig eno16777736 192.168.1.102 broadcast 192.168.1.255 netmask 255.255.255.0
#installing default gw 192.168.1.1 for vs-dr
/sbin/route add default gw 192.168.1.1
#showing routing table
/bin/netstat -rn
#checking if DEFAULT_GW 192.168.1.1 is reachable
ping -c 1 192.168.1.1
 
#set_realserver_ip_forwarding to OFF (1 on, 0 off).
echo "0" >/proc/sys/net/ipv4/ip_forward
cat       /proc/sys/net/ipv4/ip_forward
 
#looking for DIP 192.168.1.101
ping -c 1 192.168.1.101
 
#looking for VIP (will be on director)
ping -c 1 192.168.1.11
 
#install_realserver_vip
/sbin/ifconfig tunl0 192.168.1.11 broadcast 192.168.1.11 netmask 0xffffffff up
#ifconfig output
/sbin/ifconfig tunl0
#installing route for VIP 192.168.1.11 on device tunl0
/sbin/route add -host 192.168.1.11 dev tunl0
#listing routing info for VIP 192.168.1.11
/bin/netstat -rn
 
#hiding interface tunl0, will not arp
echo "1" >/proc/sys/net/ipv4/conf/all/hidden
cat       /proc/sys/net/ipv4/conf/all/hidden
echo "1" >/proc/sys/net/ipv4/conf/tunl0/hidden
cat       /proc/sys/net/ipv4/conf/tunl0/hidden
 
#----------mini-rc.lvs_dr-realserver------------------
```

分别给脚本附加执行权限

```
chmod -x lvs-tun-xxxxx.sh
```

分别运行3个脚本

```
./lvs-tun-xxxx.sh
```

TUN测试结果
![TUN结果图](https://1csh1.github.io/img/LVS的3种负载均衡技术的测试/TUN结果图.jpg)

TUN脚本文件：
[TUN脚本文件](https://1csh1.github.io/file/LVS的3种负载均衡技术的测试/tun.rar)


**第三种负载均衡：DR**
![DR搭建图](https://1csh1.github.io/img/LVS的3种负载均衡技术的测试/DR搭建图.jpg)

在node1中创建脚本：lvs-dr-node1.sh

```
#!/bin/bash
#---------------mini-rc.lvs_dr-director------------------------
#set ip_forward OFF for lvs-dr director (1 on, 0 off)
#(there is no forwarding in the conventional sense for LVS-DR)
cat       /proc/sys/net/ipv4/ip_forward
echo "0" >/proc/sys/net/ipv4/ip_forward
 
#director is not gw for realservers: leave icmp redirects on
echo 'setting icmp redirects (1 on, 0 off) '
echo "1" >/proc/sys/net/ipv4/conf/all/send_redirects
cat       /proc/sys/net/ipv4/conf/all/send_redirects
echo "1" >/proc/sys/net/ipv4/conf/default/send_redirects
cat       /proc/sys/net/ipv4/conf/default/send_redirects
echo "1" >/proc/sys/net/ipv4/conf/eno16777736/send_redirects
cat       /proc/sys/net/ipv4/conf/eno16777736/send_redirects
 
#setup DIP
/sbin/ifconfig eno16777736 192.168.1.101 broadcast 192.168.1.255 netmask 255.255.255.0
 
#add ethernet device and routing for VIP 192.168.1.11
/sbin/ifconfig eno16777736:0 192.168.1.11 broadcast 192.168.1.11 netmask 255.255.255.255
/sbin/route add -host 192.168.1.11 dev eno16777736:0
#listing ifconfig info for VIP 192.168.1.11
/sbin/ifconfig eno16777736:0
 
#check VIP 192.168.1.11 is reachable from self (director)
/bin/ping -c 1 192.168.1.11
#listing routing info for VIP 192.168.1.11
/bin/netstat -rn
 
#setup_ipvsadm_table
#clear ipvsadm table
/sbin/ipvsadm -C
#installing LVS services with ipvsadm
#add telnet to VIP with round robin scheduling
/sbin/ipvsadm -A -t 192.168.1.11:80 -s rr
 
#forward telnet to realserver using direct routing with weight 1
/sbin/ipvsadm -a -t 192.168.1.11:80 -r 192.168.1.102:80 -g -w 1
#check realserver reachable from director
ping -c 1 192.168.1.102
 
#forward telnet to realserver using direct routing with weight 1
/sbin/ipvsadm -a -t 192.168.1.11:80 -r 192.168.1.103:80 -g -w 1
#check realserver reachable from director
ping -c 1 192.168.1.103
 
#displaying ipvsadm settings
/sbin/ipvsadm
 
#not installing a default gw for LVS_TYPE vs-dr
#---------------mini-rc.lvs_dr-director------------------------
```

在node2中创建脚本：lvs-dr-node2.sh

```
#!/bin/bash
#----------mini-rc.lvs_dr-realserver------------------
#setup IP
/sbin/ifconfig eno16777736 192.168.1.102 broadcast 192.168.1.255 netmask 255.255.255.0
#installing default gw 192.168.1.1 for vs-dr
/sbin/route add default gw 192.168.1.1
#showing routing table
/bin/netstat -rn
#checking if DEFAULT_GW 192.168.1.1 is reachable
ping -c 1 192.168.1.1
 
#set_realserver_ip_forwarding to OFF (1 on, 0 off).
echo "0" >/proc/sys/net/ipv4/ip_forward
cat       /proc/sys/net/ipv4/ip_forward
 
#looking for DIP 192.168.1.101
ping -c 1 192.168.1.101
 
#looking for VIP (will be on director)
ping -c 1 192.168.1.11
 
#install_realserver_vip
/sbin/ifconfig lo:0 192.168.1.11 broadcast 192.168.1.11 netmask 0xffffffff up
#ifconfig output
/sbin/ifconfig lo:0
#installing route for VIP 192.168.1.11 on device lo:0
/sbin/route add -host 192.168.1.11 dev lo:0
#listing routing info for VIP 192.168.1.11
/bin/netstat -rn
 
#hiding interface lo:0, will not arp
echo "1" >/proc/sys/net/ipv4/conf/all/hidden
cat       /proc/sys/net/ipv4/conf/all/hidden
echo "1" >/proc/sys/net/ipv4/conf/lo/hidden
cat       /proc/sys/net/ipv4/conf/lo/hidden
 
#----------mini-rc.lvs_dr-realserver------------------
```

在node3中创建脚本：lvs-dr-node3.sh

```
#!/bin/bash
#----------mini-rc.lvs_dr-realserver------------------
#setup IP
/sbin/ifconfig eno16777736 192.168.1.103 broadcast 192.168.1.255 netmask 255.255.255.0
#installing default gw 192.168.1.1 for vs-dr
/sbin/route add default gw 192.168.1.1
#showing routing table
/bin/netstat -rn
#checking if DEFAULT_GW 192.168.1.1 is reachable
ping -c 1 192.168.1.1
 
#set_realserver_ip_forwarding to OFF (1 on, 0 off).
echo "0" >/proc/sys/net/ipv4/ip_forward
cat       /proc/sys/net/ipv4/ip_forward
 
#looking for DIP 192.168.1.101
ping -c 1 192.168.1.101
 
#looking for VIP (will be on director)
ping -c 1 192.168.1.11
 
#install_realserver_vip
/sbin/ifconfig lo:0 192.168.1.11 broadcast 192.168.1.11 netmask 0xffffffff up
#ifconfig output
/sbin/ifconfig lo:0
#installing route for VIP 192.168.1.11 on device lo:0
/sbin/route add -host 192.168.1.11 dev lo:0
#listing routing info for VIP 192.168.1.11
/bin/netstat -rn
 
#hiding interface lo:0, will not arp
echo "1" >/proc/sys/net/ipv4/conf/all/hidden
cat       /proc/sys/net/ipv4/conf/all/hidden
echo "1" >/proc/sys/net/ipv4/conf/lo/hidden
cat       /proc/sys/net/ipv4/conf/lo/hidden
 
#----------mini-rc.lvs_dr-realserver------------------
```

分别给脚本附加执行权限

```
chmod -x lvs-dr-xxxxx.sh
```

分别运行3个脚本

```
./lvs-dr-xxxx.sh
```

DR测试结果

![DR结果图](https://1csh1.github.io/img/LVS的3种负载均衡技术的测试/DR结果图.jpg)

DR脚本文件：
[TUN脚本文件](https://1csh1.github.io/file/LVS的3种负载均衡技术的测试/dr.rar)

参考文章链接：
[LVS配置教程](https://www.cnblogs.com/oldjiang/archive/2013/02/01/LVS-Nat-Tun-Dr.html)