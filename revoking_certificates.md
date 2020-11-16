# 吊销证书
吊销证书 意味着使先前签署的证书无效，从而使其不再可用于身份验证。

想要撤销证书的典型原因包括：

* 与证书关联的私钥已被破坏或被盗。
* 加密私钥的用户会忘记密钥上的密码。
* 您要终止VPN用户的访问。

### 例如
我们将撤消 文章HOW TO的key generation章节中生成的 client2证书。

首先打开一个shell或cmd，然后 像上面key generation章节中一样的cd到easy-rsa目录。
* 在Linux / BSD / Unix上：
```
. ./vars
./revoke-full client2
```
* 在Windows:
```
vars
revoke-full client2
```
您应该看到类似于以下的输出：
```
Using configuration from /root/openvpn/20/openvpn/tmp/easy-rsa/openssl.cnf
DEBUG[load_index]: unique_subject = "yes"
Revoking Certificate 04.
Data Base Updated
Using configuration from /root/openvpn/20/openvpn/tmp/easy-rsa/openssl.cnf
DEBUG[load_index]: unique_subject = "yes"
client2.crt: /C=KG/ST=NA/O=OpenVPN-TEST/CN=client2/emailAddress=me@myhost.mydomain
error 23 at 0 depth lookup:certificate revoked
```


注意最后一行中的“error 23”。它表明对已撤销证书的证书验证失败,这就是您要看到的内容。

该 `revoke-full`脚本会在keys子目录下,产生被称为crl.pem的CRL（证书吊销列表）文件。应将文件复制到OpenVPN服务器可以访问的目录，然后应在服务器配置中启用CRL验证：
```
crl-verify crl.pem
```

然后，所有连接的客户端都将根据CRL验证其客户端证书，并且任何匹配,都将导致连接断开。

### CRL说明
当 `crl-verify` 选项在OpenVPN的使用，
一旦一个新的客户端连接或现有的客户端重新协商SSL/TLS连接（默认情况下每小时一次）,CRL文件会被重新读取。这意味着您可以在OpenVPN服务器守护程序运行时更新CRL文件，并使新的CRL对新连接的客户端立即生效。如果要吊销其证书的客户端已经连接了，则可以通过信号（SIGUSR1或SIGHUP）重新启动服务器并刷新所有客户端，或者可以远程登录到 管理接口并显式杀死服务器上的特定客户端实例对象，而无需打扰其他客户。

尽管 crl-verify 指令可以在OpenVPN服务器和客户端上使用，但是通常无需向客户端分发CRL文件.除非服务器证书已撤销.否则。客户端不需要知道其他已被吊销的客户端证书，因为 客户端一开始就不应该接受来自其他客户端的直接连接。

CRL文件不是秘密文件，应使其具有全球可读性，以便OpenVPN守护程序可以在删除root privileges后读取它。

如果使用 chroot 指令，请确保将CRL文件的副本放在chroot目录中，因为与OpenVPN读取的大多数其他文件不同，CRL文件将在执行chroot调用之后而不是之前读取。

需要吊销证书的常见原因是用户使用密码对私钥进行加密，然后忘记了密码。通过吊销原始证书，可以使用 用户的原始公用名生成一个新的证书/密钥对。