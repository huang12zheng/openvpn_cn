# 设置自己的证书颁发机构（CA）

### 总览

建立OpenVPN 2.x配置的第一步是建立PKI（公钥基础结构）。PKI包括：

* 服务器和每个客户端的单独的证书（也称为公钥）和私钥，以及
* 用于对每个服务器和客户端证书进行签名的证书和密钥,证书和密钥来自于主证书颁发机构（CA）。
OpenVPN支持基于证书的双向身份验证.这意味着客户端必须对服务器证书进行身份验证，并且服务器必须在建立相互信任之前对客户端证书进行身份验证。

服务器和客户端都对彼此进行身份验证是通过如下方式:
* 首先验证提供的证书已由主证书颁发机构（CA）签名，
* 然后通过测试现在已认证的证书头中的信息（例如通用证书名称或证书类型(客户端或服务器）)。

<details>
  <summary> 从VPN角度来看，此安全模型具有许多理想的功能：</summary>
  <pre>desirable features 理想的功能</pre>
</details>


* 服务器只需要自己的证书/密钥-它不需要知道可能连接到它的每个客户端的单独证书。

<details>
  <summary>只有由主CA签名的客户端证书（我们将在下面生成）,服务器才接受.而且由于服务器可以执行此签名验证而无需访问CA私钥本身，因此CA密钥（整个PKI中最敏感的密钥）有可能驻留在完全不同的计算机上，甚至没有网络连接的计算机也可以驻留。 
  </summary>
  <blockcode> 
     The server will only accept clients whose certificates were signed by the master CA certificate (which we will generate below). And because the server can perform this signature verification without needing access to the CA private key itself, it is possible for the CA key (the most sensitive key in the entire PKI) to reside on a completely different machine, even one without a network connection.
  </blockcode>
</details>  


* 如果私钥遭到破坏，可以通过将其证书添加到CRL（证书吊销列表）来禁用它。CRL允许有选择地拒绝受到破坏的证书，而无需重建整个PKI。
* 服务器可以基于嵌入式证书字段（例如“通用名称”）强制执行特定于客户端的访问权限。

> 请注意，服务器和客户端时钟需要大致同步，否则证书可能无法正常工作。

### 生成主证书颁发机构（CA）证书和密钥

在本部分中，我们将为三个单独的客户端生成一个主CA证书/密钥，一个服务器证书/密钥以及证书/密钥。

<details>
  <summary></summary>
  <blockcode>
  For PKI management, we will use easy-rsa 2, a set of scripts which is bundled with OpenVPN 2.2.x and earlier. If you’re using OpenVPN 2.3.x, you need to download easy-rsa 2 separately from here.
  </blockcode>
</details>

* 对于PKI管理，我们将使用 easy-rsa 2，这是与OpenVPN 2.2.x和更早版本捆绑在一起的一组脚本。
* 如果您使用的是OpenVPN 2.3.x，则可能需要从easy-rsa-old项目页面单独下载easy-rsa 2。
> 在OpenVPN软件库中，还为Debian和Ubuntu提供了easy-rsa 2软件包 。
> 
> 在*NIX平台上，您应该考虑使用 easy-rsa 3
>
> 有关详细信息，请参阅其有关的文档。

* 如果您使用的是Linux，BSD或类似Unix的操作系统，请打开shell并使用cd进入 `easy-rsa` 子目录。
* 如果您是从RPM或DEB文件安装的OpenVPN，则easy-rsa目录通常可以在 `/usr/share/doc/packages/openvpn`或`/usr/share/doc/openvpn`中找到.
> 最好将此目录复制到另一个位置,例如`/etc/openvpn`.然后再进行任何编辑，以便将来的OpenVPN软件包升级不会覆盖您的修改。

* 如果是从.tar.gz文件安装的，则easy-rsa目录将位于扩展的源树的顶级目录中。

* 如果使用的是Windows，请打开“命令提示符”窗口，然后将`cd`转到`\ Program Files\OpenVPN\easy-rsa`。运行以下批处理文件以将配置文件复制到位（这将覆盖所有先前存在的vars.bat和openssl.cnf文件）：

```bat
init-config
```

现在，编辑 vars 文件（ 在Windows上称为 vars.bat）并设置KEY_COUNTRY，KEY_PROVINCE，KEY_CITY，KEY_ORG和KEY_EMAIL参数。请勿将任何这些参数留空。

接下来，初始化PKI。
* 在Linux / BSD / Unix上：

```sh
. ./vars
./clean-all
./build-ca
```

在Windows上：
```
vars
clean-all
build-ca
```

最终的命令（build-ca）将通过调用交互式openssl命令来构建证书颁发机构（CA）证书和密钥 ：

```sh
ai:easy-rsa # ./build-ca
Generating a 1024 bit RSA private key
............++++++
...........++++++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [KG]:
State or Province Name (full name) [NA]:
Locality Name (eg, city) [BISHKEK]:
Organization Name (eg, company) [OpenVPN-TEST]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:OpenVPN-CA
Email Address [me@myhost.mydomain]:
```
请注意，按照上述顺序，大多数查询的参数默认为vars或 vars.bat 文件中设置的值 。必须明确输入的唯一参数是"Common Name"。在上面的示例中，我使用了"OpenVPN-CA"。
### 生成服务器的证书和密钥
接下来，我们将为服务器生成证书和私钥。在Linux / BSD / Unix上：

```
./build-key-server server
```
在Windows上：

```
build-key-server server
```

与上一步一样，大多数参数都可以默认设置。当查询"Common Name"时，输入“server”。另外两个查询需要肯定的答复：

`Sign the certificate? [y/n]` 和

`1 out of 1 certificate requests certified, commit? [y/n]`

为3个客户端生成证书和密钥
生成客户端证书与上一步非常相似。在Linux / BSD / Unix上：

```
./build-key client1
./build-key client2
./build-key client3
```

在Windows上：
```
build-key client1
build-key client2
build-key client3
```
如果要用密码保护客户端密钥，请替换 build-key-pass 脚本。

请记住，对于每个客户端，请确保 在出现提示时键入适当的 公用名，即“ client1”，“ client2”或“ client3”。始终为每个客户端使用唯一的通用名称。

### 生成Diffie Hellman参数
 必须为OpenVPN服务器生成Diffie Hellman参数。在Linux / BSD / Unix上：
```
./build-dh
```
在Windows上：
```
build-dh
```
输出：

```
ai:easy-rsa # ./build-dh
Generating DH parameters, 1024 bit long safe prime, generator 2
This is going to take a long time
.................+...........................................
...................+.............+.................+.........
......................................
```


### 密钥文件
现在，我们将在keys 子目录中找到我们新生成的密钥和证书。这是有关文件的说明：

|文件名	|需要者|	目的	|保密|
|-|-|-|-|
|ca.crt|服务器+所有客户端|根CA证书|没有|
|ca.key|仅限密钥签名机|根CA密钥|是|
|dh{n}.pem|仅服务器|Diffie Hellman参数|没有|
|server.crt|仅服务器|服务器证书|没有|
|server.key|仅服务器|服务器密钥|是|
|client1.crt|仅client1|Client1证书|没有|
|client1.key|仅client1|Client1密钥|是|
|client2.crt|仅client2|Client2证书|没有|
|client2.key|仅client2|Client2密钥|是|
|client3.crt|仅client3|Client3证书|没有|
|client3.key|仅client3|Client3密钥|是|
密钥生成过程的最后一步是将所有文件复制到需要它们的计算机上，并注意通过安全通道复制机密文件。

但是，您可能会说。在没有预先存在的安全通道的情况下，是否可以设置PKI？
<details>
<summary>
答案明显上是可以的。出于简洁起见，在上面的示例中，我们在同一位置生成了所有私钥。我们如果再多花点心思，可以做些不同的选择。例如，我们可以让客户端在本地生成自己的私钥，而不是在服务器上生成客户端证书和密钥，然后将证书签名请求（CSR）提交给`密钥签名计算机`。反过来，`密钥签名机`可能已经处理了CSR，并将签名的证书返回给客户端。可以完成此操作，而无需密秘的 .key 文件离开生成它的计算机的硬盘驱动器。
</summary>
<blockcode>
The answer is ostensibly yes. In the example above, for the sake of brevity, we generated all private keys in the same place. With a bit more effort, we could have done this differently. For example, instead of generating the client certificate and keys on the server, we could have had the client generate its own private key locally, and then submit a Certificate Signing Request (CSR) to the key-signing machine. In turn, the key-signing machine could have processed the CSR and returned a signed certificate to the client. This could have been done without ever requiring that a secret .key file leave the hard drive of the machine on which it was generated.
</blockcode>
</details>

