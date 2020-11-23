# Easy-RSA 3快速入门教程
这是使用Easy-RSA版本3的快速入门指南。可以通过运行./easyrsa -h找到有关用法和特定命令的详细帮助。其他文档可以在doc /目录中找到。

如果要从Easy-RSA 2.x系列进行升级，则也有可用的[升级说明](EasyRSA-Upgrade-Notes.md)。

## 设置并签署第一个请求

这是快速启动新的PKI并签署您的第一个实体证书所需的操作：

1. 选择一个系统作为您的CA并创建一个新的PKI和CA：
```
./easyrsa init-pki
./easyrsa build-ca
```
2. 在请求证书的系统上，初始化其自己的PKI并生成密钥对/请求。请注意，只有在单独的系统（或至少是单独的PKI目录）上完成此操作时，才使用init-pki。这是建议的过程。如果您不使用此建议的过程，请跳过下一个import-req步骤。
```
./easyrsa init-pki
./easyrsa gen-req EntityName
```
3. 将请求（.req文件）传输到CA系统并导入。此处给出的名称是任意的，仅用于命名请求文件。
```
./easyrsa import-req /tmp/path/to/import.req EntityName
```
4. 将请求签名为正确的类型。本示例使用客户端类型：
```
./easyrsa sign-req client EntityName
```
5. 将新签署的证书传输到请求实体。除非该实体具有先前的副本，否则可能还需要CA证书（ca.crt）。

6. 该实体现在拥有自己的密钥对，签名证书和CA。

## 签署后续请求
请按照上面的步骤2-6生成后续密钥对，并使CA返回签名证书。

## 吊销证书并创建CRL
这是CA特定的任务。

要永久吊销已颁发的证书，请提供导入期间使用的简称：
```
./easyrsa revoke EntityName
```
要创建一个更新的CRL，其中包含到那时为止所有已撤销的证书：
```
./easyrsa gen-crl
```
生成后，将需要将CRL发送到引用它的系统。

## 生成Diffie-Hellman（DH）参数
初始化PKI后，任何实体都可以创建需要它们的DH参数。通常仅由TLS服务器使用。虽然CA PKI可以生成此文件，但在服务器本身上执行此操作更有意义，以避免在生成文件后将文件发送到另一个系统。

DH参数可以通过以下方式生成：
```
./easyrsa gen-dh
```
## 显示请求或证书的详细信息
要通过引用简短的EntityName显示请求或证书的详细信息，请使用以下命令之一。在没有匹配文件的情况下调用它们是错误的。
```
./easyrsa show-req EntityName
./easyrsa show-cert EntityName
```
## 更改私钥密码
RSA和EC私钥可以重新加密，因此可以根据密钥类型使用以下命令之一提供新的密码短语：
```
./easyrsa set-rsa-pass EntityName
./easyrsa set-ec-pass EntityName
```
（可选）可以使用“ nopass”标志将密码短语完全删除。请参阅命令帮助以获取详细信息。