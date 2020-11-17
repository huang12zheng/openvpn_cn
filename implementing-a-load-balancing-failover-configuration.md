# 实施负载平衡/故障转移配置

* 客户

OpenVPN客户端配置可以引用多个服务器进行负载平衡和故障转移。例如：

```
remote server1.mydomain
remote server2.mydomain
remote server3.mydomain
```

将指示OpenVPN客户端按该顺序尝试与server1，server2和server3建立连接。如果现有连接断开，则OpenVPN客户端将重试最近连接的服务器，如果失败，将移至列表中的下一个服务器。您还可以指示OpenVPN客户端在启动时随机分配其服务器列表，以便客户端负载可以在服务器池中随机分布。

```
remote-random
```
如果您还希望DNS解析失败导致OpenVPN客户端移动到列表中的下一个服务器，请添加以下内容：

```
resolv-retry 60
```
在 60 个参数告诉OpenVPN客户端来尝试解决每个 远程 列表上移动到下一个服务器前60秒DNS名称。

服务器列表还可以引用在同一台计算机上运行的多个OpenVPN服务器守护程序，每个守护程序侦听不同端口上的连接，例如：

```
remote smp-server1.mydomain 8000
remote smp-server1.mydomain 8001
remote smp-server2.mydomain 8000
remote smp-server2.mydomain 8001
```
如果您的服务器是多处理器计算机，则从性能角度来看，在每台服务器上运行多个OpenVPN守护程序可能是有利的。

OpenVPN还支持 引用DNS名称的 远程指令，该DNS名称 在域的区域配置中具有多个 A记录。在这种情况下， 每次解析域时，OpenVPN客户端都会随机选择A记录之一 。

* 服务器
在服务器上进行负载平衡/故障转移配置的最简单方法是在群集中的每个服务器上使用等效的配置文件，但为每个服务器使用不同的虚拟IP地址池。例如：

服务器1
```
server 10.8.0.0 255.255.255.0
```
服务器2
```
server 10.8.1.0 255.255.255.0
```
服务器3
```
server 10.8.2.0 255.255.255.0
```