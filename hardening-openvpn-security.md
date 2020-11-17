# 加强OpenVPN安全性

网络安全性常被反复提及的准则之一是，永远不要对单个安全性组件如此信任，以至于它的故障会导致灾难性的安全性破坏。OpenVPN提供了几种机制来添加额外的安全层来对付这种结果。

* 身份验证
TLS-auth的指令增加了额外的HMAC签名,来对所有的SSL/TLS握手进行完整性验证。任何没有正确HMAC签名的UDP数据包都可以丢弃，而无需进一步处理。所述TLS-AUTHHMAC签名提供超出由SSL/TLS提供了额外的安全水平。它可以防止：

OpenVPNUDP端口上的DoS攻击或端口泛洪。
端口扫描以确定哪些服务器UDP端口处于侦听状态。
SSL/TLS实现中的缓冲区溢出漏洞。
来自未经授权的机器的SSL/TLS握手启动（虽然这种握手最终将无法进行身份验证，但tls-auth可以在更早的时候切断它们）。
使用tls-auth要求您生成除标准RSA证书/密钥之外还使用的共享密钥：
```
openvpn --genkey --secret ta.key
```
此命令将生成一个OpenVPN静态密钥，并将其写入文件ta.key。该密钥应通过预先存在的安全通道复制到服务器和所有客户端计算机。可以将其与RSA.key和.crt文件放在同一目录中。

在服务器配置中，添加：
```
tls-auth ta.key 0
```
在客户端配置中，添加：
```
tls-auth ta.key 1
```
* proto udp

尽管OpenVPN允许将TCP或UDP协议用作VPN运营商连接，但与TCP相比，UDP协议将提供更好的DoS攻击和端口扫描保护：
```
proto udp
```
user/group (仅非Windows)
OpenVPN经过精心设计，允许在初始化后删除root特权，并且该功能应始终在Linux/BSD/Solaris上使用。如果没有root特权，运行中的OpenVPN服务器守护程序将为攻击者提供诱人的目标。

```
user nobody
group nobody
```
* 非特权模式Unprivileged mode (仅Linux)

在Linux上，OpenVPN可以完全无特权地运行。此配置稍微复杂一点，但是提供了最佳的安全性。

为了使用此配置，必须将OpenVPN配置为使用iproute接口，这是通过指定–enable-iproute2来配置脚本来完成的。sudo软件包也应该在您的系统上可用。

此配置使用Linux功能来更改tun设备的权限，以便非特权用户可以访问它。它还使用sudo来执行iproute，以便可以修改接口属性和路由表。

OpenVPN配置：

编写以下脚本并将其放在：/usr/local/sbin/unpriv-ip：
```
#!/bin/sh
sudo /sbin/ip $*
```
执行visudo，并添加以下内容以允许用户'user1'执行/sbin/ip：
```
user1 ALL=(ALL)  NOPASSWD: /sbin/ip
```
您还可以使用以下命令启用一组用户：
```
%users ALL=(ALL)  NOPASSWD: /sbin/ip
```
将以下内容添加到您的OpenVPN配置中：
```
dev tunX/tapX
iproute /usr/local/sbin/unpriv-ip
```
请注意，您必须选择常数X并指定tun或不选择两者。
作为root添加持久性接口，并允许用户和/或组对其进行管理，以下创建tunX（替换为您自己的）并允许user1和group用户访问它。
```
openvpn --mktun --dev tunX --type tun --user user1 --group users
```
在非特权用户的上下文中运行OpenVPN。

通过检查/usr/local/sbin/unpriv-ip脚本中的参数，可以添加其他安全约束。

* chroot（仅非Windows）

在chroot的指令允许你指定OpenVPN锁定到一个所谓的chroot监狱，那里的守护进程将无法访问主机系统的文件系统的任何部分，除了作为参数的指令中的特定目录。例如，

```
chroot jail
```
会导致OpenVPN守护程序在初始化时进入jail子目录，然后将其根文件系统重新定向到此目录，以便此后守护程序无法访问jail及其子目录树之外的任何文件。从安全角度来看，这很重要，因为即使攻击者能够使用代码插入漏洞破坏服务器，该漏洞也将被锁定在服务器大部分文件系统之外。

注意事项：因为chroot重定向了文件系统（仅从守护程序的角度来看），所以有必要将初始化后OpenVPN可能需要的所有文件放在jail目录中，例如：

crl-verify文件，或
client-config-dir 目录。

* 较大的RSA密钥

<details>
<summary>
RSA密钥大小由easy-rsa/vars文件中的KEY_SIZE变量控制，必须在生成任何密钥之前对其进行设置。当前默认情况下设置为1024，该值可以合理地增加到2048，而对VPN隧道性能没有负面影响，除了SSL/TLS重新协商握手会稍微慢一些(每小时每个客户端一次的).重新协商握手比使用easy-rsa/build-dh脚本来进行的Hellman参数生成过程慢得多
</summary>
<blockcode>
The RSA key size is controlled by the KEY_SIZE variable in the easy-rsa/vars file, which must be set before any keys are generated. Currently set to 1024 by default, this value can reasonably be increased to 2048 with no negative impact on VPN tunnel performance, except for a slightly slower SSL/TLS renegotiation handshake which occurs once per client per hour, and a much slower one-time Diffie Hellman parameters generation process using the easy-rsa/build-dh script.
</blockcode>
</details>


* 更大的对称密钥

默认情况下，OpenVPN使用 Blowfish（128位对称密码）。

OpenVPN自动支持OpenSSL库支持的任何密码，因此可以支持使用大密钥大小的密码。例如，可以通过将以下内容添加到服务器和客户端配置文件来使用256位版本的AES（高级加密标准）：

```
cipher AES-256-CBC
```

* 将根密钥（ca.key）保留在没有网络连接的独立计算机上

使用X509 PKI（就像OpenVPN一样）的安全性好处之一是，根CA密钥（ca.key）不需要出现在OpenVPN服务器计算机上。在高安全性环境中，您可能希望专门为密钥签名目的指定一台计算机，对计算机进行良好的物理保护，并使其与所有网络断开连接。根据需要，可以使用软盘来回移动密钥文件。这样的措施使攻击者极难窃取根密钥，而密钥签名机的物理失窃却很少。