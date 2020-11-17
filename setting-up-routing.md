# 路由配置

如果设置了路由VPN，即本地和远程子网不同，则需要在子网之间设置路由，以便数据包能够通过VPN。

这是可能的公路战士网络配置：
* Road Warrior (Windows)
```
TAP-Windows Adapter
10.3.0.2 subnet 255.255.255.0
```

ifconfig option in OpenVPN config:
```
ifconfig 10.3.0.2 255.255.255.0
```
* Main Office, server (any OS)
```
tap adapter
10.3.0.1 subnet 255.255.255.0
```
ifconfig option in OpenVPN config:
```
ifconfig 10.3.0.2 255.255.255.0
private ethernet
10.0.0.1 subnet 255.255.255.0
```

The road warrior needs this route in order to reach machines on the main office subnet:

```
route add 10.0.0.0 mask 255.255.255.0 10.3.0.1 (this is a shell command)
```
Routes can be conveniently specified in the OpenVPN config file itself using the –route option:
```
route 10.0.0.0 255.255.255.0 10.3.0.1
```



如果总公司中的OpenVPN服务器也是远程子网中计算机的网关，则总公司中不需要特殊的路由。

另一方面，如果主办公室OpenVPN服务器也不是网关，则作为网关的任何计算机或路由器都必须知道将10.3.0.0子网255.255.255.0路由 到运行OpenVPN的计算机。