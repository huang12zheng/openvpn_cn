# 如果客户端未验证所连接服务器的证书，则可能发生的“中间人”攻击的重要说明。

为避免可能的中间人攻击，在这种情况下，授权客户端会通过模拟服务器尝试连接到另一个客户端，请确保强制客户端执行某种服务器证书验证。当前有五种不同的方法可以完成此操作，按优先顺序列出：

* [OpenVPN 2.1及更高版本]使用特定密钥用法和扩展密钥用法来构建服务器证书。RFC3280确定应为TLS连接提供以下属性：
<table>
<tbody>
<tr>
<th>模式</th>
<th>Key 用法</th>
<th>Extended key 用法</th>
</tr>
<tr>
<td rowspan="3">Client</td>
<td>digitalSignature数字签名</td>
<td rowspan="3">TLS Web Client Authentication</td>
</tr>
<tr>
<td>keyAgreement密钥协商</td>
</tr>
<tr>
<td>digitalSignature, keyAgreement</td>
</tr>
<tr>
<td rowspan="2">Server</td>
<td>digitalSignature, keyEncipherment密钥加密</td>
<td rowspan="2">TLS Web Server Authentication</td>
</tr>
<tr>
<td>digitalSignature, keyAgreement</td>
</tr>
</tbody></table>

您可以使用build-key-server 脚本来构建服务器证书 （有关更多信息，请参见 easy-rsa文档）。通过设置正确的属性，这会将证书指定为仅服务器证书。请将以下行添加到客户端配置：

```
remote-cert-tls server
```
* [OpenVPN 2.0及以下版本] 使用build-key-server 脚本构建服务器证书 （ 有关更多信息，请参见 easy-rsa文档）。通过设置nsCertType = server，这会将证书指定为仅服务器证书。请将以下行添加到客户端配置：

```
ns-cert-type server
```

这将阻止客户端连接到证书中缺少nsCertType = server名称的任何服务器 ，即使证书已由 OpenVPN配置文件中的ca文件签名也是如此.

* 在客户端上使用 tls-remote指令，可以基于服务器证书的通用名称来接受/拒绝服务器连接.

<details> 
<summary>根据 嵌入式X509 的详细信息和服务器证书,发起一个自定义测试.来实现使用tls-verify脚本或插件,去接受/拒绝服务器连接。
</summary>
<blockcode>
 Use a tls-verifyscript or plugin to accept/reject the server connection based on a custom test of the server certificate’s embedded X509 subject details.
</blockcode>
</details>

* 使用一个CA签署服务器证书，并使用不同的CA签署客户端证书。客户端配置 ca 指令应引用服务器签名CA文件，而服务器配置 ca 指令应引用客户端签名CA文件。