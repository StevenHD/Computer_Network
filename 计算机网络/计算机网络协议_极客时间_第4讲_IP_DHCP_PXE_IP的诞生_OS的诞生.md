# 第4讲 | DHCP与PXE：IP是怎么来的，又是怎么没的？

## 配置IP

发送 ARP 请求，获取 MAC 地址。

IP ，给网卡配置这么一个地址。

包发不出去，这是因为 MAC 层还没填。

只有是一个网段的，它才会发送 ARP 请求，获取 MAC 地址。

Linux 默认的逻辑是，如果这是一个跨网段的调用，它便不会直接将包发送到网络上，而是企图将包发送到网关。

## 动态主机配置协议（DHCP）

一个自动配置IP地址的协议，也就是动态主机配置协议（Dynamic Host Configuration Protocol），简称 DHCP。

DHCP 的方式就相当于租房。

- DHCP Discover
- DHCP Offer——给新人分配的地址

新来的机器使用 IP 地址 0.0.0.0 发送了一个广播包，目的 IP 地址为 255.255.255.255。

当一台机器带着自己的 MAC 地址加入一个网络的时候，MAC 是它唯一的身份

当 DHCP Server 接收到客户机的 DHCP request 之后，会广播返回给客户机一个 DHCP ACK 消息包，表明已经接受客户机的选择。

---

## IP 地址的收回和续租

## 预启动执行环境（PXE）

没安装系统之前，连启动扇区都没有。因而这个过程叫做预启动执行环境（Pre-boot Execution Environment），简称 PXE。

PXE 协议分为客户端和服务器端。

PXE 的工作过程。

- 首先，启动 PXE 客户端，DHCP Server 便租给它一个 IP 地址
- PXE 客户端向 TFTP 服务器请求下载启动文件 pxelinux.0，TFTP 服务器说好啊，于是就将这个文件传给它。
- PXE 客户端收到这个文件后，就开始执行这个文件。
- 最后，启动 Linux 内核。

---

##### DHCP 协议主要是用来给客户租用 IP 地址

##### DHCP 协议能给客户推荐“装修队”PXE，能够安装操作系统，这个在云计算领域大有用处。