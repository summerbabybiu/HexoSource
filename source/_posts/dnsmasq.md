---
title: dnsmasq 的使用手册
date: 2018-09-13 15:02:13
tags: dnsmasq
---

如何更好的用手机调试本地的页面、接口？
我选择在本地搭建了 DNS 服务器。
dnsmasq 轻量且易配置，提供 DNS 缓存和 DHCP 服务功能。

<!--more-->

# 安装

使用 brew 直接安装
```
$ brew install dnsmasq
```
# 配置

### 配置 `/usr/local/etc/dnsmasq.conf`
以下我提供的是最常见的配置

- resolv-file
指定 dns 解析的文件, 默认是 /etc/resolv.conf
```
resolv-file=/usr/local/etc/resolv.dnsmasq.conf 
```

- strict-order
表示严格按照 resolv-file 文件中的顺序从上到下进行 DNS 解析, 直到第一个成功解析成功为止

- listen-address
监听地址，172.17.9.78为局域网分配给我本机的IP，如果只填写127.0.0.1，那么只针对本机有效，局域网内的其他服务器则无法访问这个 DNS 服务器。
```
listen-address=172.17.9.78,127.0.0.1
```

- no-hosts
dnsmasq 首先会寻找本地的 /etc/hosts 文件，再去寻找缓存下来的域名, 最后去上游 dns 服务器寻找。此项将禁止 dnsmasq 访问 本地 hosts

- addn-hosts
自定义其他的 hosts 文件，按需求看自己要不要配哈，不配的话，会默认访问 /etc/hosts
```
addn-hosts=/usr/local/etc/dnsmasq.hosts
```

我的配置情况如下：
```
resolv-file=/usr/local/etc/resolv.dnsmasq.conf 
listen-address=172.17.9.78,127.0.0.1
strict-order
```

### 配置 `resolv-file`

```
$ sudo vi /usr/local/etc/resolv.dnsmasq.conf 
```
文件内容如下：

```
nameserver 127.0.0.1 // 本机
nameserver 114.114.114.114
nameserver 1.1.1.1
nameserver 172.17.40.160 // 局域网本身的 DNS （因为我的页面需要请求内网接口，所以需要加上这行）
```

### 配置 `/usr/local/etc/dnsmasq.hosts` 或 `/etc/hosts`

```
172.17.9.78 hello.me
```
如果配成 127.0.0.1 hello.me，局域网其他服务器无法访问 hello.me 这个域名

启动 dnsmasq

```
# 启动
sudo brew services start dnsmasq

# 重启
sudo brew services restart dnsmasq

# 停止
sudo brew services stop dnsmasq
```

如果改动了泛解析规则，重启 dnsmasq 不会立即看到效果，可尝试清楚缓存
```
sudo killall -HUP mDNSResponder
```

# 测试

- ping 一下 hello.me，看能不能 Ping 通
- 本机打开网页，看能不能正常访问网络。
- 手机在于电脑连接同一个局域网的条件下，配置 wifi 的 DNS，DNS 地址为电脑的 IP，然后尝试在手机打开 hello.me, 看能否正常访问。

测试成功的话，就可以愉快的在手机上调试本地的开发页面啦~~~
