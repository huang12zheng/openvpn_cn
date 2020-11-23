Easy-RSA高级教程
============================

这是熟悉PKI流程的高级用户的技术参考。如果
您需要更详细的说明，请参阅[EasyRSA-Readme](EasyRSA-Readme.md)或[Intro-To-PKI](Intro-To-PKI.md)
文档代替。

配置参考
-----------------------

####配置源

  有3种可能的方式来执行Easy-RSA的外部配置,
  按照第一个定义的结果获胜的顺序选择：

  1. 命令行选项
  2. 环境变量
  3. "vars"文件（如果存在）（请参见下面的"vars自动检测"）
  4. 内置默认

  请注意，并非所有可能的配置选项都可以在任何地方设置，尽管任何
  即使默认情况下未显示，也可以将env-var添加到"vars"文件中。

#### vars自动检测

 "vars"文件是一个名为"vars"（无扩展名）的文件，
  Easy-RSA将提供配置信息。该文件是专门设计的
  *不能*替换已使用较高优先级方法设置的变量
  例如CLI opts或env-vars。

  按照以下顺序检查以下位置的vars文件。只有
  使用的第一个被使用：

  1. CLI选项`--vars` 引用的文件
  2. 用"EASYRSA_VARS_FILE"引用env-var文件
  3. 用`EASYRSA_PKI` 引用env-var目录
  4. 默认的PKI目录位于$ PWD / pki中。
  4. 用`EASYRSA` 引用env-var目录
  5. 包含easyrsa程序的目录

  定义env-var`EASYRSA_NO_VARS`将覆盖vars的来源
  在所有情况下都包含文件，包括随后将其定义为全局选项。

#### OpenSSL配置

  Easy-RSA与OpenSSL配置文件（.cnf）紧密耦合，用于
  脚本提供的灵活性。要求此文件可用，
  但是可以为特定的文件使用不同的OpenSSL配置文件
  PKI，甚至针对特定调用更改它。

  按照以下顺序搜索OpenSSL配置文件：

  1. env-var`EASYRSA_SSL_CONF`
  2. "vars"文件（请参见上面的"vars自动检测"）
  3. "EASYRSA_PKI"目录下，文件名为"openssl-easyrsa.cnf"
  4. "EASYRSA"目录下，文件名为"openssl-easyrsa.cnf"

高级扩展处理
---------------------------

通常，证书扩展是通过CLI上给出的证书类型选择的
签名期间；这将导致x509-types子目录中的匹配文件
被处理以添加OpenSSL扩展。这可以在
通过在EASYRSA_PKI目录内放置另一个x509类型的目录来实现特定的PKI
它将代替使用。

x509-types目录中名为"COMMON"的文件将附加到每种证书类型；
这是专为CDP使用而设计的，但可以用于任何应该
适用于每个签名证书。

另外，env-var`EASYRSA_EXTRA_EXTS`的内容后面附加
其原始文本已添加到OpenSSL扩展中。内容按原样附加到
证书扩展名；无效的OpenSSL配置通常会导致失败。

环境变量参考
---------------------------------

env-vars列表，任何设置/覆盖它的匹配全局选项（CLI）以及
可能的简短描述如下所示：

 * `EASYRSA`-应该指向Easy-RSA顶级目录，在easyrsa
    脚本位于。
 * `EASYRSA_OPENSSL`-调用openssl的命令
 * `EASYRSA_SSL_CONF`-要使用的openssl配置文件
 * `EASYRSA_PKI`（CLI：`--pki-dir`）-用于保存所有特定于PKI的目录
    文件，默认为`$ PWD / pki`。
 * `EASYRSA_DN`（CLI：`--dn-mode`）-设置为字符串`cn_only`或`org`
    更改要包括在请求DN中的字段
 * `EASYRSA_REQ_COUNTRY`（CLI：`--req-c`）-使​​用组织模式设置DN国家
 * `EASYRSA_REQ_PROVINCE`（CLI：`--req-st`）-使用以下命令设置DN状态/省
    组织模式
 * `EASYRSA_REQ_CITY`（CLI：`--req-city`）-使用org设置DN城市/位置
    模式
 * `EASYRSA_REQ_ORG`（CLI：`--req-org`）-用org模式设置DN组织
 * `EASYRSA_REQ_EMAIL`（CLI：`--req-email`）-使用组织模式设置DN电子邮件
 * `EASYRSA_REQ_OU`（CLI：`--req-ou`）-使用org设置DN组织单位
    模式
 * `EASYRSA_KEY_SIZE`（CLI：`--key-size`）-将密钥大小设置为
    生成
 * `EASYRSA_ALGO`（CLI：`--use-algo`）-设置要使用的加密算法：rsa或ec
 * `EASYRSA_CURVE`（CLI：`--curve`）-定义要使用的命名EC曲线
 * `EASYRSA_EC_DIR`-存储生成的ecparams的目录
 * `EASYRSA_CA_EXPIRE`（CLI：`--days）-以天为单位设置CA过期时间
 * `EASYRSA_CERT_EXPIRE`（CLI：`--days）-设置颁发的证书到期时间
    在几天内
 * `EASYRSA_CRL_DAYS`（CLI：`--days）-以天为单位设置CRL的"下一次发布"时间
 * `EASYRSA_NS_SUPPORT`（CLI：`--ns-cert`）-字符串'yes'或'no'到
    包括已弃用的Netscape扩展
 * `EASYRSA_NS_COMMENT`（CLI：`--ns-comment`）-字符串注释，包括何时
    使用不推荐使用的Netscape扩展
 * `EASYRSA_TEMP_FILE`-动态创建req / cert时要使用的临时文件
     扩展名
  * `EASYRSA_REQ_CN`（CLI：`--req-cn`）-默认CN，需要在BATCH中设置
     模式
  * `EASYRSA_DIGEST`（CLI：`--digest`）-设置哈希摘要以用于req / cert
     签署
  * `EASYRSA_BATCH`（CLI：`--batch`）-启用批处理（无提示）模式； 组
     env-var为非零字符串以启用（CLI不带任何选项）
  * `EASYRSA_PASSIN`（CLI：`--passin`）-允许为
     使用任何openssl密码选项（例如pass：1234或env：var）输入密码
  * `EASYRSA_PASSOUT`（CLI：`--passout`）-允许为
     使用任何openssl密码选项（例如pass：1234或env：var）输入密码
    
**注意：**在实际命令之前需要提供全局选项。