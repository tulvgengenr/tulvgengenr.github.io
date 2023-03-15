---
title: "SSH端口转发"
description: "SSH三种端口转发方式：本地端口转发、远程端口转发和动态端口转发"
pubDate: "March 14 2023"
heroImage: "/post_forward.webp"
---
在项目中遇到了一个问题：需要在A结点和B结点之间实现ssh免密登录，但是A结点和B结点属于两个不同的子网。而C结点有两块网卡，既可以访问A结点所在的子网，又可以访问B结点所在的子网，因此可以通过端口转发的方式，将C结点作为跳板，实现A与B之间的ssh免密。步骤如下：

1. 首先实现C到A、B的免密：  
```shell
# C shell
ssh-copy-id HostB
ssh-copy-id HostA
```

2. 选择**本地端口转发**，使用`-L [bind_address:]port:host:hostport`命令：  
```shell
# A shell
# -f:后台执行   -g:允许远程主机连接到本地转发端口   -N:不执行远程命令通常与-f连用   -L:本地端口转发
ssh -fgNL [HostA]:22222:HostB:22 HostC
```  
```shell
# B shell
ssh -fgNL [HostB]:22222:HostA:22 HostC
```  

==注意：-N 选项（可选）表示这个 SSH 连接只进行端口转发，不登录远程 Shell，不能执行远程命令，只能充当隧道；==

这样，A结点只需要`ssh -p 22222 HostA `就可以实现免密登录B结点了，同样的B结点只需要`ssh -p 22222 HostB`就可以实现免密登录A结点。

总之，SSH端口转发通过将客户端或服务器端的某个端口的数据放到 SSH 连接中加密传输，从而**在不安全的网络中安全地传输数据**或者**绕过防火墙的限制实现对目的主机目的端口的访问**。 

实际上除了上面的本地端口转发，还有远程端口转发和动态端口转发两种方式。  


## 本地端口转发

本地端口转发 是指在 SSH 连接的基础上，将远程的 SSH 服务器作为中介，建立一个本地客户端主机到目的主机（第三台主机）的连接，将对于本地客户端主机指定端口的访问通过 SSH 连接转发给 SSH 服务器，再由 SSH 服务器发到目标主机的目标端口，同时将结果传回。此时转发的是本地端口。

## 远程端口转发

通常采用`-R [bind_address:]port:host:hostport`命令，同样是上面的案例，我们想从A结点访问B结点。执行以下命令：  
```shell
# A shell
ssh -fNR 22222:HostB:22 HostC
```  
此时转发的端口是将C结点的22222端口进行了转发，当从A结点想要访问B结点时，可以直接访问C结点的22222端口，即`ssh -p 22222 HostC`

### 常用场景

其中有一种常用场景是，如果被访问服务器没有公网ip，可以通过转发另外一个有公网ip的服务器端口从而提供访问服务。

```sh
# 在没有公网ip的服务器上执行
ssh -R 80:localhost:80 cloud_user@server.com
```

此时可以访问公网域名server.com的http服务即访问了没有公网ip的服务器的http服务。

## 动态端口转发

转发的入端口是本地端口，但是出端口是远程服务器的**不固定**的端口，具体访问远程服务器的什么服务，取决于发起的请求。在这种情况下，可以将本地机器制作成一个代理服务器。

命令格式如下:

```sh
# 本地shell
ssh -D [-fN] 本地端口 user@host
```

使用实例：

B为跨国服务器，A为本地服务器，现在要将A制作为代理服务器，并用C访问。

```sh
# A shell
ssh -D 2223 root@HostB
```

```sh
# C shell
curl -x socks5://HostA:2223 http://www.google.com
```
上面命令中，curl的-x参数指定代理服务器，socks5为代理的协议。