# 通过OpenVPN连接到Samba共享

此示例旨在说明OpenVPN客户端如何通过路由的开发 隧道连接到Samba共享 。如果您是以太网桥接（dev tap），则可能不需要遵循这些说明，因为OpenVPN客户端应在其网络邻居中看到服务器端计算机。

对于此示例，我们将假定：

* 服务器端LAN使用的子网是 10.66.0.0/24，
* VPN IP地址池使用 10.8.0.0/24  （如 OpenVPN服务器配置文件中的 server指令中所述），
* Samba服务器的IP地址为 10.66.0.4，并且
* Samba服务器已经配置好，可以从本地LAN访问。
* 如果Samba和OpenVPN服务器在不同的机器上运行，请确保已按照 将VPN的范围扩展到包括其他机器的部分进行操作。

接下来，编辑您的Samba配置文件（smb.conf）。确保 hosts allow 指令将允许来自10.8.0.0/24 子网的OpenVPN客户端 进行连接。例如：
```
hosts allow = 10.66.0.0/24 10.8.0.0/24 127.0.0.1
```
如果您在同一台计算机上运行Samba和OpenVPN服务器，则可能需要编辑smb.conf 文件中的 interfaces 指令， 来在10.8.0.0/24的TUN接口子网上也进行监听：

```
interfaces  = 10.66.0.0/24 10.8.0.0/24
```
如果您在同一台计算机上运行Samba和OpenVPN服务器，请使用文件夹名称从OpenVPN客户端连接到Samba共享：

```
\\10.8.0.1\\sharename
```
如果Samba和OpenVPN服务器在不同的计算机上，请使用文件夹名称：
```
\\10.66.0.4\sharename
```
例如，在命令提示符窗口中：
```
net use z: \\10.66.0.4\sharename /USER:myusername
```